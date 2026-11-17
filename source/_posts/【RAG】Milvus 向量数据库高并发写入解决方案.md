---
title: 【RAG】Milvus 向量数据库高写入读取解决方案
date: 2026-11-10 15:38:14
tags:
    - RAG
    - Milvus
---

# Milvus 向量数据库高并发写入解决方案

在 Milvus 向量数据库中，应对**高并发读取（Read-Heavy）**，核心不是简单地增加 QueryNode 数量，而是从查询架构、数据分片、副本机制、缓存、索引优化、请求调度和资源隔离等多个层面进行优化。

如果你的业务场景是 RAG 知识库问答，例如多个用户同时发起向量检索，那么需要重点解决以下问题：

1. 大量并发 Search 请求导致 QueryNode CPU、内存资源耗尽。
2. 向量检索耗时增加，P95、P99 延迟升高。
3. 多个用户查询相同知识库，重复计算造成资源浪费。
4. 大规模向量数据导致索引检索性能下降。
5. 高并发读取与数据写入相互影响。
6. 突发流量导致请求堆积，甚至出现超时和服务雪崩。

## 一、整体架构设计

采用：**接入层负载均衡 + 检索服务集群 + Milvus 多副本 + 多级缓存**。

整体请求链路：

```
                 用户请求
                    |
                    v
              API Gateway
                    |
          限流 / 鉴权 / 熔断
                    |
                    v
              检索服务集群
          +---------+---------+
          |         |         |
       Search-1  Search-2  Search-3
          |         |         |
          +---------+---------+
                    |
                    v
              Redis 缓存
                    |
              Cache Miss
                    |
                    v
              Milvus Proxy
                    |
                    v
             Query Coordinator
                    |
          +---------+---------+
          |         |         |
       QueryNode QueryNode QueryNode
          |         |         |
          +---------+---------+
                    |
               Segment数据
```

**注意**：Milvus 内部已经有 Proxy、QueryCoord、QueryNode 等组件，应用层不应该绕过 Milvus 自己实现底层分片路由，而是通过 SDK 或 REST API 发起查询。

## 二、核心优化策略

### 2.1 增加 QueryNode 副本，提高并发读取能力

优先级：最高

这是 Milvus 应对高并发查询最重要的手段之一。

Milvus 支持通过 Replica 机制，将 Collection 的数据副本分布到多个 QueryNode 上。

例如：

```
                  Milvus Proxy
                       |
              Query Coordinator
                       |
          +------------+------------+
          |            |            |
       Replica-1    Replica-2    Replica-3
          |            |            |
       QueryNode    QueryNode    QueryNode
          |            |            |
       数据副本      数据副本      数据副本
```

当多个用户同时发起 Search 请求时，Milvus 可以将查询分发到不同的副本，利用更多计算资源处理请求。

**具体实现：**

```python
from pymilvus import Collection

collection = Collection("knowledge_base")

# 创建 3 个副本
collection.load(replica_number=3)

# 查看副本信息
replicas = collection.get_replicas()

print(replicas)
```

**注意**：

- 增加副本会增加内存、CPU 和数据加载资源消耗。
- 副本数不能超过集群可提供的合理资源容量。
- 副本需要有足够的 QueryNode 资源承载。
- 增加副本不意味着单次查询一定更快，主要改善并发吞吐量。

生产环境可以从 2～3 个副本开始压测，根据 CPU、内存、P95 延迟和 QPS 决定是否继续扩展。

### 2.2 使用 Redis 缓存热点查询结果

优先级：最高

在 RAG 场景中，很多用户会查询相同或相似的问题。

例如：

- 什么是高血压？
- 高血压有哪些症状？
- 高血压患者需要注意什么？

如果每次都执行 Milvus Search，会产生大量重复计算。

因此可以在 Milvus 前面增加 Redis 缓存。

```
            用户 Query
                 |
                 v
          Embedding Model
                 |
                 v
           生成 Query Key
                 |
                 v
             Redis
             /    \
          Hit      Miss
           |         |
           v         v
        返回结果   Milvus Search
                     |
                     v
                 Redis缓存
                     |
                     v
                  返回结果
```

```python
import redis
import json
import hashlib
import numpy as np

from pymilvus import Collection

redis_client = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)

collection = Collection("knowledge_base")


def generate_cache_key(query_vector, top_k):
    vector_bytes = np.asarray(
        query_vector, dtype=np.float32
    ).tobytes()

    vector_hash = hashlib.sha256(
        vector_bytes
    ).hexdigest()

    return f"milvus:search:{vector_hash}:{top_k}"


def search_vectors(query_vector, top_k=5):
    cache_key = generate_cache_key(
        query_vector, top_k
    )

    # 1. 查询缓存
    cached = redis_client.get(cache_key)

    if cached:
        return json.loads(cached)

    # 2. 缓存未命中，查询 Milvus
    results = collection.search(
        data=[query_vector],
        anns_field="embedding",
        param={
            "metric_type": "COSINE",
            "params": {"nprobe": 16}
        },
        limit=top_k,
        output_fields=["text", "document_id"]
    )

    # 3. 结果转换
    response = [
        {
            "id": hit.id,
            "distance": hit.distance,
            "text": hit.entity.get("text")
        }
        for hit in results[0]
    ]

    # 4. 写入 Redis，设置过期时间
    redis_client.setex(
        cache_key,
        300,
        json.dumps(response, ensure_ascii=False)
    )

    return response
```

生产环境还需要注意：

- Cache Key 应包含租户、知识库、权限范围、Embedding 模型版本、索引版本等信息。
- 权限过滤条件必须参与缓存 Key，防止跨用户数据泄露。
- 对缓存设置 TTL，避免知识库更新后返回旧数据。
- 使用缓存击穿保护，避免热点 Key 过期时大量请求同时访问 Milvus。

**注意**：向量检索缓存应该优先考虑相同向量的精确命中。如果希望缓存语义相似但不完全相同的问题，需要额外设计近似查询缓存，不能直接把语义相似当作结果等价。

### 2.3 优化向量索引，降低单次查询计算成本

增加副本解决的是并发吞吐量问题，优化索引解决的是单次查询的计算成本问题。

Milvus 常用索引：

| 索引       | 特点              | 适用场景        |
| -------- | --------------- | ----------- |
| HNSW     | 图结构近似搜索，查询速度快   | 高性能 RAG 检索  |
| IVF_FLAT | 倒排聚类，搜索时只扫描部分聚类 | 大规模向量检索     |
| IVF_PQ   | 向量压缩，降低内存占用     | 超大规模向量库     |
| FLAT     | 暴力精确搜索          | 数据量小、要求精确召回 |
| DISKANN  | 利用磁盘存储索引        | 数据规模超过内存容量  |

对于 RAG 场景，采用 HNSW 或 IVF_FLAT。

以 HNSW 为例：

```python
index_params = {
    "index_type": "HNSW",
    "metric_type": "COSINE",
    "params": {
        "M": 16,
        "efConstruction": 200
    }
}

collection.create_index(
    field_name="embedding",
    index_params=index_params
)
```

查询时：

```python
search_params = {
    "metric_type": "COSINE",
    "params": {
        "ef": 64
    }
}

results = collection.search(
    data=[query_vector],
    anns_field="embedding",
    param=search_params,
    limit=10,
    output_fields=["text"]
)
```

关键参数：

- `M`：影响 HNSW 图的连接数量，越大通常召回能力越好，但内存消耗和构建成本也越高。
- `efConstruction`：控制索引构建质量。
- `ef`：控制查询时搜索范围，越大通常召回率越高，但查询耗时也会增加。

**注意**：索引参数不是越大越好，应结合召回率和 P99 延迟进行压测。

### 2.4 合理设置分片和 QueryNode 数量

Milvus 中需要区分两个概念：

- Shard：数据分片，主要用于将数据分布到不同的计算节点。
- Replica：数据副本，主要用于提高并发读取能力和可用性。

例如：

```
Collection
   |
   +--- Shard 1
   |       |
   |       +--- Replica 1
   |       +--- Replica 2
   |
   +--- Shard 2
   |       |
   |       +--- Replica 1
   |       +--- Replica 2
   |
   +--- Shard 3
           |
           +--- Replica 1
           +--- Replica 2
```

两者解决的问题不同：

| 优化方式         | 主要目的          |
| ------------ | ------------- |
| 增加 Shard     | 提高数据分布和并行处理能力 |
| 增加 Replica   | 提高读取吞吐量       |
| 增加 QueryNode | 增加计算资源        |
| 增加 CPU       | 提高单节点计算能力     |

对于 RAG 知识库，如果数据量不是特别大，但是并发查询量非常高，优先考虑增加 Replica，而不是盲目增加 Shard。

### 2.5 优化查询参数和返回字段

高并发时，不能忽略单次查询的工作量。

例如：

```python
results = collection.search(
    data=[query_vector],
    anns_field="embedding",
    param={
        "metric_type": "COSINE",
        "params": {"ef": 64}
    },
    limit=10,
    expr='tenant_id == 1001',
    output_fields=[
        "document_id",
        "text"
    ]
)
```

重点优化：

**第一，控制 TopK。**

如果 RAG 只需要召回 10～20 个分块，就不要每次返回 100～500 条。

**第二，减少返回字段。**

向量检索阶段尽量只返回 ID、必要的元数据和少量文本，避免返回大量冗余字段。

**第三，使用标量过滤。**

例如按照：

- tenant_id
- knowledge_base_id
- document_type
- permission_level

进行过滤。

对经常用于过滤的字段建立合适的标量索引，减少无效数据扫描。

**第四，控制批量查询大小。**

不要将大量用户请求无限制地合并为一个超大的 `nq` 请求。批量查询可以提升吞吐量，但也可能导致单个任务占用过多资源，影响其他请求的尾延迟。


### 2.6 合理选择一致性级别

Milvus 支持四种一致性级别：

| 一致性        | 特点           | 使用场景       |
| ---------- | ------------ | ---------- |
| Strong     | 强一致，等待最新数据可见 | 对实时性要求极高   |
| Bounded    | 有界延迟，默认级别    | 普通知识库检索    |
| Session    | 保证当前会话写入可见   | 用户上传后立即查询  |
| Eventually | 最终一致，允许一定延迟  | 对数据新鲜度要求较低 |

在高并发读取场景下，如果业务不要求每次查询都读取最新数据，不建议全部使用 Strong。

例如：

```python
results = collection.search(
    data=[query_vector],
    anns_field="embedding",
    param=search_params,
    limit=10,
    consistency_level="Bounded"
)
```

**注意**：一致性级别影响的是数据可见性和等待时间，不是简单的查询加速开关。


### 2.7 应用层限流、并发控制和连接池

即使 Milvus 集群拥有充足的计算资源，也不能让应用层无限制地发起请求。

在检索服务层增加：

- 连接复用：避免每次查询都创建新连接。
- 并发限制：控制同时执行的 Search 数量。
- 超时控制：避免请求长期占用资源。
- 熔断降级：Milvus 故障时快速失败。
- 请求排队：对短时间突发流量进行缓冲。
- 多租户限流：避免单个租户占满整个检索集群。


```python
import asyncio
from pymilvus import Collection

collection = Collection("knowledge_base")

# 限制同时执行的检索任务数量
semaphore = asyncio.Semaphore(100)


async def search(query_vector, top_k=10):

    async with semaphore:

        results = await asyncio.wait_for(
            asyncio.to_thread(
                collection.search,
                data=[query_vector],
                anns_field="embedding",
                param={
                    "metric_type": "COSINE",
                    "params": {"ef": 64}
                },
                limit=top_k,
                output_fields=["text"]
            ),
            timeout=3
        )

        return results
```

这里的 `asyncio.to_thread` 是为了避免同步 Milvus SDK 阻塞事件循环，并不意味着 Milvus 查询本身变成了异步执行。

生产环境还应该：

1. 统一管理 Milvus Client 或 Collection 连接，避免每个请求重复初始化。
2. 对线程池大小进行限制。
3. 为不同知识库或租户设置独立并发配额。
4. 超时后正确取消上层任务，并防止无限制积累后台工作线程。
5. 根据压测结果调整 Semaphore，而不是固定使用 100。

## 三、生产环境架构：读写分离与资源隔离

如果系统不仅有高并发读取，还有大量文档上传、向量化、写入，那么还需要考虑读写资源竞争。

例如 RAG 系统：

```
                    API Gateway
                         |
                  限流 / 鉴权
                         |
                 +-------+-------+
                 |               |
              查询请求         写入请求
                 |               |
                 v               v
             Search API       Ingestion API
                 |               |
                 v               v
              Redis           Kafka
                 |               |
             Cache Miss          v
                 |           Embedding
                 |               |
                 v               v
             Milvus Proxy <--- Insert
                 |
                 v
           Query Coordinator
                 |
          +------+------+
          |             |
       QueryNode      QueryNode
       Replica 1      Replica 2
          |             |
          +------+------+
                 |
              查询结果
```

实际生产部署中，采用 Milvus Distributed 模式，通过 Kubernetes 分别管理相关组件的资源。

特别是：

- QueryNode：重点保障搜索 CPU、内存和磁盘 I/O。
- DataNode：避免索引构建、数据处理任务干扰查询。
- Streaming Node：保障实时数据写入与流式查询。
- Proxy：根据接入流量扩容。

## 四、监控 Milvus 的读取性能

接入 Prometheus + Grafana，重点关注以下指标。

| 指标                  | 关注点         | 异常表现     |
| ------------------- | ----------- | -------- |
| Search QPS          | 每秒查询量       | 吞吐量达到瓶颈  |
| Search P95          | 95% 请求耗时    | 用户体验下降   |
| Search P99          | 99% 请求耗时    | 长尾延迟严重   |
| QueryNode CPU       | CPU 使用率     | 计算资源不足   |
| QueryNode Memory    | 内存使用率       | 内存压力过大   |
| Query Queue Latency | 请求排队耗时      | 并发超过处理能力 |
| Search Error Rate   | 检索失败率       | 超时或资源不足  |
| Cache Hit Rate      | Redis 缓存命中率 | 缓存策略需要优化 |

尤其需要关注：

- `milvus_querynode_sq_queue_latency`：QueryNode 查询任务排队耗时。
- `milvus_querynode_sq_core_latency`：查询核心执行耗时。
- `milvus_querynode_sq_reduce_latency`：查询结果归并耗时。
- `milvus_proxy_sq_latency`：Proxy 查询耗时。

通过这些指标，可以判断问题究竟出现在请求排队、向量搜索还是结果归并阶段。

## 五、如何进行高并发压测？

使用 Locust 或 JMeter 进行压测。

压测时固定：

- 向量数据规模。
- Embedding 模型。
- TopK。
- 索引类型及参数。
- 过滤条件。
- 返回字段。
- 一致性级别。

逐步增加并发：

| 并发用户 | 重点观察     |
| ---- | -------- |
| 10   | 基准性能     |
| 50   | 初步并发能力   |
| 100  | CPU、内存变化 |
| 200  | QPS 增长趋势 |
| 500  | 是否出现排队   |
| 1000 | 是否出现性能拐点 |

这里的并发用户只是压测梯度，不代表 Milvus 的实际承载能力。

最终重点看三个指标：

1. QPS：吞吐量是否持续增长。
2. P99：尾延迟是否突然上升。
3. Error Rate：是否出现超时和失败。

当并发继续增加，但 QPS 不再明显增长，同时 P99 快速上升时，通常就已经接近系统的饱和点。


## 六、最终生产环境优化方案

RAG 知识库问答，按照下面的优先级实施：

| 优先级 | 优化手段              | 解决问题         |
| --- | ----------------- | ------------ |
| P0  | Redis 热点缓存        | 减少重复查询       |
| P0  | QueryNode Replica | 提高并发吞吐       |
| P0  | 应用层限流和并发控制        | 防止请求过载       |
| P1  | HNSW / IVF 索引优化   | 降低单次查询耗时     |
| P1  | TopK、过滤条件优化       | 减少无效计算       |
| P1  | 连接复用              | 降低连接管理开销     |
| P1  | 监控及自动扩缩容          | 提高系统稳定性      |
| P2  | 读写资源隔离            | 避免写入影响读取     |
| P2  | 多租户资源隔离           | 防止热点租户影响其他用户 |

**我认为最值得优先实施的是：**

**Redis + Milvus Replica + 应用层并发控制**

1. Redis 缓存热点查询，减少 Milvus 实际请求量。
2. Milvus 配置多个 Replica，利用多个 QueryNode 分担读取请求。
3. 检索服务通过连接复用、Semaphore、超时控制限制并发。
4. 通过压测确定集群吞吐量上限，再根据 P99 和 CPU 使用率扩容。
   