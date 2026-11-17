---
title: 【RAG】Milvus 向量数据库高并发写入解决方案
date: 2026-11-10 15:38:14
tags:
    - RAG
    - Milvus
---

# Milvus 向量数据库高并发写入解决方案

Milvus 在面对高并发写入时，核心问题并不只是提高 Insert 请求的吞吐量，而是要解决：
- 写入流量削峰
- 数据写入吞吐
- 资源隔离
- 批量持久化
- 数据一致性以及写入与查询之间的资源竞争

在企业级 RAG 知识库场景中，同时上传 PDF、Word、Excel 文档，经过解析、分块、Embedding 向量化后写入 Milvus，如果直接由业务服务并发调用 Insert，很容易出现：

1. 大量小批次 Insert，导致请求开销和调度开销增加。
2. Embedding 服务生成速度过快，Milvus 写入能力跟不上。
3. DataNode、QueryNode 等组件资源竞争，影响查询性能。
4. 数据持续写入导致 Flush、Compaction 压力增大。
5. 多个客户端同时写入相同数据，出现重复数据或业务幂等问题。
6. 高峰期写入积压，导致内存压力、请求超时甚至服务不可用。


为应对以上问题，采用如下架构：
**Kafka 消息队列 + 批量写入 + 异步消费 + Milvus 分布式集群 + 写入限流**



## 一、整体架构设计

文档处理与向量写入解耦，避免业务请求直接承受 Milvus 的写入压力。

```
             用户上传文档
                  |
                  v
          文档解析服务
                  |
                  v
             文本分块
                  |
                  v
          Embedding 服务
                  |
                  v
          向量数据生产者
                  |
                  v
        +-------------------+
        |       Kafka       |
        |  消息缓冲 / 削峰   |
        +-------------------+
                  |
                  v
          向量写入调度服务
                  |
          +-------+-------+
          |               |
          v               v
      Batch Worker    Batch Worker
          |               |
          +-------+-------+
                  |
                  v
           Milvus Proxy
                  |
                  v
          Milvus DataCoord
                  |
                  v
             DataNode
                  |
                  v
         Object Storage
                  |
                  v
          Flush / Compaction

                  |
                  v
          Milvus QueryNode
                  |
                  v
             RAG 检索
```

**注意**：Milvus 内部采用组件化架构，Proxy 接收客户端请求，DataCoord 管理数据协调，DataNode 负责数据持久化相关处理，底层数据最终存储在对象存储中。Kafka 是业务侧缓冲层，不应与 Milvus 内部消息流机制混淆。

## 二、六个核心优化策略

### 2.1 Kafka 削峰填谷：解决瞬时高并发

**解决问题**：系统在某一时刻有 1000 个用户同时上传文档，每个文档经过切分产生 500 个 Chunk，那么短时间可能产生 50 万条向量数据。如果直接向 Milvus 发起 50 万次写入请求，会造成较大的请求压力。

**解决方案**：

- 文档解析和 Embedding 完成后，将向量数据写入 Kafka。
- 根据业务场景设置 Topic 和 Partition。
- 写入消费者根据 Milvus 当前负载动态调整消费速度。
- 当 Milvus 写入能力不足时，Kafka 暂存积压数据。
- 当 Milvus 恢复后，消费者继续消费。

示例：
```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=["localhost:9092"],
    value_serializer=lambda x: json.dumps(
        x, ensure_ascii=False
    ).encode("utf-8"),
    acks="all",
    retries=10,
    enable_idempotence=True
)

def publish_vectors(vectors):
    producer.send(
        "milvus_vector_write",
        value={
            "vectors": vectors
        }
    )
    producer.flush()
```

**核心价值：** 将瞬时流量转换成持续、可控的写入流量。

### 2.2 批量写入：减少请求开销

**解决问题**：大量小批次 Insert，导致请求开销和调度开销增加

例如：

| 写入方式     | 请求次数   | 说明        |
| -------- | ------ | --------- |
| 单条写入     | 100000 | 请求开销大     |
| 每100条批量  | 1000   | 有明显改善     |
| 每500条批量  | 200    | 减少请求开销    |
| 每1000条批量 | 100    | 进一步降低请求次数 |

**解决方案**：两种触发策略

- 数量触发：达到 500～1000 条时执行批量写入。
- 时间触发：等待 100～500ms，即使数量不足也执行写入。

具体阈值需要通过压测确定，不能认为批次越大越好，因为过大的批次也会增加内存占用和请求延迟。

示例：
```Python
from pymilvus import (
    connections,
    Collection
)
import time

connections.connect(
    alias="default",
    host="localhost",
    port="19530"
)

collection = Collection("knowledge_vectors")

BATCH_SIZE = 500
FLUSH_INTERVAL = 0.3

buffer = []
last_flush_time = time.time()

def insert_batch(batch):
    if not batch:
        return

    collection.insert(batch)

def add_vector(vector_data):
    global last_flush_time

    buffer.append(vector_data)

    current_time = time.time()

    if (
        len(buffer) >= BATCH_SIZE
        or current_time - last_flush_time >= FLUSH_INTERVAL
    ):
        batch = buffer.copy()
        buffer.clear()

        insert_batch(batch)
        last_flush_time = current_time
```


### 2.3 控制并发度：避免写入请求压垮 Milvus

**解决问题**：假设有 20 个 Worker，每个 Worker 同时发起 10 个 Insert 请求，那么 Milvus 同时可能承受 200 个写入请求。如果超过集群的处理能力，就会出现：

- gRPC 请求超时。
- DataNode 或 StreamingNode 资源紧张。
- 内存使用率持续上升。
- 写入延迟不断增加。
- Compaction 和索引构建任务积压。

**解决方案**：
引入一个并发调度器。

示例代码：

```
Kafka
  |
  v
Consumer Group
  |
  v
并发调度器
  |
  +---- Worker 1 ----+
  +---- Worker 2 ----+
  +---- Worker 3 ----+----> Milvus
  +---- Worker 4 ----+
  +---- Worker 5 ----+
```

可以使用信号量控制最大并发写入数：

```python
import asyncio
from pymilvus import AsyncMilvusClient

client = AsyncMilvusClient(
    uri="http://localhost:19530"
)

MAX_CONCURRENCY = 5
semaphore = asyncio.Semaphore(MAX_CONCURRENCY)

async def write_batch(batch):
    async with semaphore:
        return await client.insert(
            collection_name="knowledge_vectors",
            data=batch
        )

async def write_all(batches):
    tasks = [
        write_batch(batch)
        for batch in batches
    ]

    return await asyncio.gather(
        *tasks,
        return_exceptions=True
    )
```

生产环境采用动态并发控制：

| Milvus 状态    | 调度策略                 |
| ------------ | -------------------- |
| CPU、内存正常，延迟低 | 逐步提高并发               |
| 写入延迟上升       | 降低并发                 |
| 请求超时增加       | 暂停扩容，减少消费            |
| Kafka 积压增加   | 根据数据库负载决定是否扩容 Worker |
| Milvus 恢复正常  | 逐步恢复消费               |

**优化点**：进一步采用 AIMD（加性增大、乘性减小）算法控制并发量。

例如：

- 连续 30 秒写入成功率正常，增加 1 个并发。
- 超时率超过阈值，并发量减半。
- 设置最大并发和最小并发，防止震荡。

这样可以避免 Milvus 因瞬时流量过高而被压垮。

### 2.4 Milvus 集群扩容：提升整体写入能力

单个 Milvus 的写入能力存在瓶颈，可以考虑集群扩容。

Milvus Distributed 主要采用组件化架构，可以根据负载情况进行资源扩展。

| 组件                    | 职责              | 优化方向         |
| --------------------- | --------------- | ------------ |
| Proxy                 | 接收客户端请求、请求路由    | 增加 Proxy 实例  |
| StreamingNode（2.6 架构） | 流式写入处理          | 根据写入负载扩容     |
| DataCoord             | 数据协调、Segment 管理 | 关注协调任务和元数据压力 |
| QueryNode             | 向量检索            | 独立扩容，保证查询能力  |
| IndexNode             | 索引构建            | 增加计算资源       |
| 对象存储                  | 持久化数据           | 提升 I/O 吞吐    |

**注意**：

1. 版本差异，Milvus 2.6 的新架构引入了 StreamingNode，不能简单地把旧版本 DataNode 的职责和扩容方式直接套用到新版本。
2. Collection 的 Shard 数量。
Milvus 会根据 Shard 将写入请求路由到对应的虚拟通道，再分配到相应的写入处理组件。因此，合理设置 Shard 数量有助于提高写入并行度。

示例代码：

```python
from pymilvus import MilvusClient

client = MilvusClient(
    uri="http://localhost:19530"
)

client.create_collection(
    collection_name="knowledge_vectors",
    dimension=768,
    metric_type="COSINE",
    shard_num=4
)
```

这里的 `shard_num=4` 只是示例，需要结合：

- 写入吞吐量。
- Collection 数据规模。
- 并发写入数量。
- 集群 StreamingNode 数量。
- 查询负载。

进行压测确定。

**注意**：不要为了提高写入并发而无限增加 Shard。 Shard 过多也会增加通道、元数据和调度管理开销。

### 2.5 Flush 和 Compaction 优化：避免频繁持久化

不要在每次 Insert 后执行：`collection.insert(data)collection.flush()`，这种方式并不适合高并发场景。

原因是频繁 Flush 会导致 Segment 封存、持久化及后续索引构建等工作更加频繁，增加系统资源开销。

方案：

1. 采用批量 Insert。
2. 让 Milvus 自己管理正常的 Segment 封存和持久化流程。
3. 不要每批数据都手动 Flush。
4. 关注 Compaction 积压情况。
5. 对历史数据采用批量导入。

**注意**：Milvus 的 Insert 返回成功，并不意味着数据已经完成持久化，也不意味着立即能够被搜索到。客户端确认、数据可见性和持久化是不同的阶段。

### 2.6 幂等性和失败重试：保证高并发下的数据正确性

高并发写入一定要考虑重复消费和失败重试。

例如：

```
Kafka 消息
    |
    v
Worker 执行 Insert
    |
    v
Milvus 写入成功
    |
    v
Worker 返回结果
    |
    X 网络异常
    |
    v
Kafka 消息重新消费
    |
    v
再次 Insert
```

这里可能产生重复数据。

**注意**：Milvus 的普通 Insert 不会因为 Primary Key 相同就自动执行更新，也不会自动保证主键唯一。


使用确定性主键：

```python
import hashlib

def generate_primary_key(
    document_id,
    chunk_index,
    version
):
    raw = (
        f"{document_id}:"
        f"{chunk_index}:"
        f"{version}"
    )

    digest = hashlib.sha256(
        raw.encode()
    ).digest()

    return int.from_bytes(
        digest[:8],
        byteorder="big"
    ) & 0x7FFFFFFFFFFFFFFF
```

生产设计：

- Document ID：文档唯一标识。
- Chunk Index：分块序号。
- Version：文档版本。
- Primary Key：通过上述字段生成稳定 ID。

对于更新操作，可以采用 Upsert，或者通过业务版本管理实现旧版本删除、新版本写入。

但要注意，Upsert 也不是所有并发更新场景下的事务保障，尤其是同一实体同时被多个请求修改时，需要在业务层进行协调。


Kafka 消息确认建议遵循：

```
消费 Kafka 消息
      |
      v
执行 Milvus 批量写入
      |
      v
确认写入成功
      |
      v
提交 Kafka Offset
```

如果写入失败，则不能直接提交 Offset，而应该进入重试机制；对于无法恢复的异常，进入死信队列。

## 三、千万级数据如何进一步优化？

如果不是实时增量写入，而是一次性导入几十万、几百万甚至上千万条向量数据，采用 Milvus Bulk Import（批量导入）。


流程：

```
       原始文档数据
             |
             v
       Embedding计算
             |
             v
       多Worker并行处理
             |
             v
       生成Parquet文件
             |
             v
       上传对象存储
             |
             v
       Milvus Bulk Import
             |
             v
       后台索引构建
             |
             v
       数据校验
             |
             v
       Collection上线
```

两种写入方式的选择：

| 场景         | 推荐方案              |
| ---------- | ----------------- |
| 实时新增文档     | Insert 批量写入       |
| 文档更新       | Upsert / 版本管理     |
| 百万级历史数据初始化 | Bulk Import       |
| 千万级、亿级离线导入 | Bulk Import       |
| 实时数据持续进入   | Kafka + 批量 Insert |


## 四、如何监控写入性能？

不能仅关注 Insert 成功率，还要监控写入链路的整体状态。

建立以下监控指标：

| 监控层    | 指标                | 作用         |
| ------ | ----------------- | ---------- |
| Kafka  | Consumer Lag      | 判断数据积压     |
| Kafka  | 消费速率              | 判断消费能力     |
| Worker | Batch Size        | 判断批量效率     |
| Worker | 并发数               | 判断资源利用率    |
| Worker | 写入失败率             | 判断异常情况     |
| Milvus | Insert Throughput | 判断写入吞吐     |
| Milvus | Insert Latency    | 判断写入延迟     |
| Milvus | CPU / Memory      | 判断资源瓶颈     |
| Milvus | Segment 数量及状态     | 判断数据组织情况   |
| Milvus | Compaction 状态     | 判断后台整理压力   |
| Milvus | Index Build 状态    | 判断索引构建是否积压 |

特别关注三个指标：

1. **写入吞吐量**：每秒能够写入多少条向量（Vectors/s）。
2. **P99 写入延迟**：99% 的写入请求能否在目标时间内完成。
3. **Kafka Consumer Lag**：消息积压是否持续增加。

如果 Kafka 积压不断增加，说明生产速度已经超过消费速度，需要判断瓶颈究竟在 Embedding、Worker，还是 Milvus。

## 五、企业级最终架构

![](data:image/png;base64,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)

## 六、实际项目中应该如何选择参数？

假设有一个知识库系统：

- 7000 名用户。
- 每个用户每天上传 10 份文档。
- 每份文档平均产生 500 个 Chunk。
- 每个 Chunk 对应一个 768 维向量。

那么：

\\[ 7000 \times 10 \times 500 = 3500万 \\]

每天需要写入约 3500 万条向量。

平均每秒写入量：

\\[ \frac{3500万}{86400} \approx 405 \\]

如果业务集中在 2 小时内完成，则平均每秒写入约 4861 条。

这意味着不能只按照日均 405 条设计，还必须考虑峰值流量。

采用以下初始配置进行压测：

| 参数              | 初始建议     |
| --------------- | -------- |
| Batch Size      | 500～1000 |
| Worker 数量       | 4～8      |
| 单 Worker 并发     | 1～2      |
| Batch 刷新间隔      | 300ms    |
| Kafka Partition | 8～16     |
| 重试次数            | 3～5      |
| 最大重试间隔          | 指数退避     |
| Flush           | 不逐批手动执行  |

这些是压测起点，不是生产环境的固定配置。最终要根据向量维度、字段大小、硬件资源和 P99 延迟进行调整。

