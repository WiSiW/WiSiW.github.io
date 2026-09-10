---
title: 【FDE】文章文本溯源系统设计
date: 2026-09-07 22:41:41
tags:
  - FDE
  - 系统设计
---



## 设计目标

> 给定一篇文章，判断其中每个关键陈述（Claim）是否能够在知识库/资料库中找到可信的原文证据，并进一步判断是“有直接文本支撑”“有间接文本支撑”“知识库没有依据”还是“疑似大模型臆造”。



## 产品最终形态

例子：

> 2024年，公司通过引入Flink实现了实时数据处理，并将系统处理延迟降低了80%。

系统不是只返回：

> 相似度：92%

这种模糊回答，而是应该返回：

| 原文陈述              | 溯源结果   | 证据                                   |
| --------------------- | ---------- | -------------------------------------- |
| 公司于2024年引入Flink | 🟢 强支撑   | 《技术架构升级报告》P12                |
| 实现实时数据处理      | 🟢 强支撑   | 《实时计算平台建设方案》P8             |
| 延迟降低80%           | 🔴 无依据   | 知识库没有找到相关数据                 |
| 2024年                | 🟡 部分支撑 | 文档只证明2024年上线，无法证明引入时间 |

最终进一步给文本一个：

> 文本可信度：78%



## 核心架构

                    ┌──────────────────┐
                    │      用户文章     │
                    └────────┬─────────┘
                             │
                             ▼
                  ┌────────────────────┐
                  │   Document Parser  │
                  │ 文档解析/结构识别     │
                  └─────────┬──────────┘
                            │
                            ▼
                  ┌────────────────────┐
                  │   Claim Extractor  │
                  │    陈述/事实拆解     │
                  └─────────┬──────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
          Claim 1        Claim 2       Claim 3
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                  ┌────────────────────┐
                  │  Query Generation  │
                  │ 多Query检索策略      │
                  └─────────┬──────────┘
                            │
            ┌───────────────┼────────────────┐
            ▼               ▼                ▼
       BM25 Search      Vector Search    Knowledge Graph
            │               │                │
            └───────────────┼────────────────┘
                            ▼
                  ┌────────────────────┐
                  │    Reranker        │
                  │   证据重排序         │
                  └─────────┬──────────┘
                            ▼
                  ┌────────────────────┐
                  │ Evidence Collector │
                  │    证据片段抽取       │
                  └─────────┬──────────┘
                            ▼
                  ┌────────────────────┐
                  │ Entailment Engine  │
                  │ Claim ↔ Evidence   │
                  └─────────┬──────────┘
                            ▼
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           Supported     Partial       Unsupported
           强支撑          部分支撑         无支撑
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                  ┌────────────────────┐
                  │ Provenance Engine  │
                  │    溯源关系构建       │
                  └─────────┬──────────┘
                            ▼
                  ┌────────────────────┐
                  │    Trace Report    │
                  │ 文章溯源分析报告      │
                  └────────────────────┘



## 系统设计

```
┌─────────────────────────────────────────┐
│             应用层                       │
│  文章溯源  │ 事实核验 │ 风险检测 │ 报告      │
├─────────────────────────────────────────┤
│             推理层                       │
│ Claim分析 │ Evidence验证 │ 冲突检测        │
├─────────────────────────────────────────┤
│             检索层                       │
│ BM25 │ Vector │ Reranker │ Graph        │
├─────────────────────────────────────────┤
│             知识层                       │
│ 原始文档 │ Chunk │ Claim │ Evidence       │
│ Document │ Page  │ Sentence │ Metadata  │
└─────────────────────────────────────────┘
```



## 核心数据链

```
Document
   ↓
Paragraph
   ↓
Sentence
   ↓
Claim
   ↓
Query
   ↓
Evidence
   ↓
Entailment
   ↓
Verification
   ↓
Provenance
   ↓
Report
```



## 关键设计

不是直接对“文章”做RAG，因为：

> 语义相似 ≠ 事实得到证明

例如：

知识库：

> 2024年，公司开始使用Flink进行实时计算

文章：

> 2024年，公司使用Flink将系统性能提升了80%

这款两个文本非常相似，Embedding可以得到：

> similarity = 0.91

但是其中：

```
公司使用Flink √

2024年 √

性能提升80% ×
```

所以必须先做陈述拆解（Claim Decomposition）

### 陈述拆解

文章：

> 公司于2024年完成数据中台升级，引入Flink实时计算框架，使数据处理延迟降低80%，同时支持每日千万级数据处理。

拆分为：

```json
[
  {
    "claim_id": "C001",
    "text": "公司于2024年完成数据中台升级",
    "type": "EVENT",
    "entities": ["公司", "数据中台"],
    "time": "2024"
  },
  {
    "claim_id": "C002",
    "text": "公司引入Flink实时计算框架",
    "type": "TECHNOLOGY",
    "entities": ["公司", "Flink"]
  },
  {
    "claim_id": "C003",
    "text": "Flink使数据处理延迟降低80%",
    "type": "METRIC",
    "value": "80%"
  },
  {
    "claim_id": "C004",
    "text": "系统支持每日千万级数据处理",
    "type": "CAPABILITY",
    "value": "千万级"
  }
]
```



### 陈述分类

分为：

```
FACT
事实

EVENT
事件

TIME
时间

PERSON
人物

ORGANIZATION
组织

LOCATION
地点

METRIC
指标

RELATION
关系

OPINION
观点

INFERENCE
推断

CONCLUSION
结论
```

不同类型的陈述，验证方法不同：

例如：

>  “公司在2024年完成架构升级。”

属于：

>  事实 + 时间 （EVENT + TIME）

需要找：

```
谁
什么时候
做了什么
```

而：

>  “这个架构具有很高的扩展性。”

属于：

>  观点 + 评价 （OPINION / EVALUATION）



不能要求知识库必须存在一模一样的事实。



## 检索层

> 混合检索（Hybrid Retrieval）

即：

                  Claim
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      BM25       Vector       Graph
        │           │           │
        ▼           ▼           ▼
     Keyword     Semantic     Entity
     Search       Search      Relation
        │           │           │
        └───────────┼───────────┘
                    ▼
                Fusion
                    │
                    ▼
                Reranker

### BM25 + Vector

例如 Claim：

> 张三于2023年加入XX公司担任CTO。

知识库原文：

> 2023年6月，张三正式加入XX科技，任首席技术官。

关键词：

```
张三
2023
加入
CTO
```

BM25很好。

但是如果原文：

> 李明于2023年开始负责公司技术战略。

语义可能相关，但实际上：

```
人物 ≠
职位 ≠
事件
```



所以 Vector Search 找到候选后必须继续验证。



### Reranker

候选证据：

```
Evidence 1
Evidence 2
Evidence 3
...
Evidence 20
```

使用 Reranker：

```
Claim
 ↓
Cross Encoder
 ↓
Evidence Score
```

例如：

```
C001
│
├── E001 score 0.96
├── E002 score 0.87
├── E003 score 0.53
└── E004 score 0.21
```

只保留 Top N。



## Entailment

不要问：

> “这两个文本像不像？”

而应该问：

> Evidence是否能够证明Claim？

可以把判断做成：

```
SUPPORTED
PARTIALLY_SUPPORTED
CONTRADICTED
NOT_SUPPORTED
```

例如：

**Claim**

> 公司在2024年引入Flink，并将处理延迟降低80%。

**Evidence**

> 公司于2024年引入Flink进行实时数据计算。

判断：

```
SUPPORTED:
    公司2024年引入Flink

NOT_SUPPORTED:
    延迟降低80%
```

所以整体：

```
PARTIALLY_SUPPORTED
```



## 设计四态而不是简单二分类

不要：

> 有依据 / 没依据

而采用：

| 状态           | 含义               |
| -------------- | ------------------ |
| 🟢 SUPPORTED    | 原文可以直接证明   |
| 🟡 PARTIAL      | 只能证明部分内容   |
| 🔴 UNSUPPORTED  | 知识库没有证据     |
| ⚠️ CONTRADICTED | 知识库存在相反证据 |

例如：

```
Claim:
2024年公司引入Flink，使延迟下降80%。

Evidence:
2024年公司引入Flink。

结果：
PARTIAL
```

这是比传统RAG高级很多的地方。



## 必须解决“数字幻觉”

对于企业文章，最危险的通常不是普通描述，而是：

```
数字
时间
比例
排名
金额
数量
性能
增长率
```

例如：

> 用户数量超过7000万。

知识库：

> 用户数量超过700万。

Embedding可能依然非常高。

但是：

```
7000万
vs
700万
```

属于严重错误。

因此需要建立：

**Numeric Consistency Checker**

提取：

```json
{
  "value": 70000000,
  "unit": "人",
  "type": "COUNT"
}
```

Evidence：

```json
{
  "value": 7000000,
  "unit": "人"
}
```

直接判断：

> ❌ CONTRADICTED



## 时间也需要单独验证

例如：

文章：

> 2022年，公司开始使用Flink。

知识库：

> 2024年，公司开始使用Flink。

Embedding：

```
0.94
```

但是：

```
Claim Time = 2022
Evidence Time = 2024
```

因此：

```
CONTRADICTED
```



所以我建议Claim结构化成：

```json
{
  "subject": "公司",
  "predicate": "使用",
  "object": "Flink",
  "time": "2022",
  "location": null,
  "value": null
}
```

Evidence同样结构化。

然后进行：

```
Subject Match
Predicate Match
Object Match
Time Match
Numeric Match
Entity Match
```



## 知识库必须保存“原始证据”

这是很多RAG系统容易犯的错误。

不要只保存：

```
chunk
embedding
```

至少保存：

```json
{
  "document_id": "DOC001",
  "document_name": "数据中台建设报告",
  "version": "v2.0",
  "page": 12,
  "section": "3.2 实时计算架构",
  "paragraph_id": "P023",
  "sentence_id": "S004",
  "text": "公司于2024年引入Flink...",
  "source_type": "REPORT",
  "publish_date": "2024-12-01",
  "author": "xxx",
  "department": "技术部"
}
```

这样才能真正做到：

> 点击证据 → 跳转原文位置。



## 证据来源需要分级

> Source Reliability Score

例如：

| 来源          | 权重 |
| ------------- | ---: |
| 正式制度/公告 | 1.00 |
| 官方报告      | 0.95 |
| 正式项目文档  | 0.95 |
| 技术设计文档  | 0.90 |
| 会议纪要      | 0.80 |
| 内部Wiki      | 0.75 |
| 用户上传文档  | 0.70 |
| 网络资料      | 0.60 |
| AI生成内容    | 0.10 |

于是：

```
Evidence Score
=
Semantic Score
×
Entailment Score
×
Source Reliability
×
Freshness
```











