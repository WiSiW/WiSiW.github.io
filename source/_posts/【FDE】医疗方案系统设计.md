---
title: 【FDE】医疗方案系统设计
date: 2026-10-03 00:11:04
tags:
     - FDE
---

# 目的

> 把医生长期积累的隐性经验，从大量真实医患对话中提炼出来，形成结构化、可审核、可追溯、可持续迭代的“医导知识体系”，最终让非医学专业的医导能够按照经过医生审核的知识回答患者常见问题。


# 解决的问题

```
医生日常与患者交流
        │
        ▼
     音频采集
        │
        ▼
      ASR转写
        │
        ▼
   对话理解/清洗
        │
        ▼
 ┌───────────────────┐
 │   医疗知识挖掘层    │
 │                   │
 │ FAQ Agent         │
 │ Intent Agent      │
 │ Medical Fact Agent│
 │ Answer Agent      │
 │ Risk Agent        │
 │ Evidence Agent    │
 └───────────────────┘
        │
        ▼
   知识整合与冲突检测
        │
        ▼
    医生审核工作台
        │
        ▼
   Approved Knowledge
        │
        ▼
   医导知识库 / RAG
        │
        ▼
 医导输入患者问题
        │
        ▼
   Guide Copilot Agent
        │
        ▼
  标准答案 + 注意事项
```

这实际上需要两个系统：

**知识生产 Agent**

负责：**从医生的历史对话中生产知识**。

**知识消费 Agent**

负责：**帮助医导使用已经审核过的知识回答患者问题**。

# 整体框架
```
┌───────────────────────────────────────────┐
│                用户交互层                 │
│ 医生端 │ 医导端 │ 管理端 │ 审核端         │
└───────────────────────────────────────────┘
                    │
┌───────────────────────────────────────────┐
│                Agent应用层                │
│                                           │
│ Knowledge Mining Agent                    │
│ Knowledge Review Agent                    │
│ Guide Copilot Agent                       │
└───────────────────────────────────────────┘
                    │
┌───────────────────────────────────────────┐
│                 Agent Flow                │
│                                           │
│ ASR → Cleaning → Topic → FAQ              │
│       → Medical Fact → Answer → Risk      │
│       → Evidence → Conflict → Synthesis   │
└───────────────────────────────────────────┘
                    │
┌───────────────────────────────────────────┐
│                  知识层                   │
│ FAQ │ Medical Fact │ Answer │ Risk        │
│ Entity │ Evidence │ Version │ Policy      │
└───────────────────────────────────────────┘
                    │
┌───────────────────────────────────────────┐
│                  数据层                   │
│ PostgreSQL │ Vector DB │ Elasticsearch    │
│ Object Storage │ Redis                   │
└───────────────────────────────────────────┘
                    │
┌───────────────────────────────────────────┐
│                 AI基础设施                │
│ ASR │ Embedding │ LLM │ Reranker          │
└───────────────────────────────────────────┘
```

# Agent体系设计

12个专业 Agent + 1个 Orchestrator。

## Orchestrator Agent

整个系统的大脑。

负责：
```
接收任务
 ↓
判断当前阶段
 ↓
调用对应Agent
 ↓
合并结果
 ↓
决定下一步
```

采用：
> 确定性 Workflow + Agent 节点。

例如：
```
Audio
 ↓
ASR Agent
 ↓
Conversation Agent
 ↓
Topic Agent
 ↓
FAQ Agent
 ↓
Medical Fact Agent
 ↓
Answer Agent
 ↓
Risk Agent
 ↓
Evidence Agent
 ↓
Conflict Agent
 ↓
Knowledge Synthesis Agent
 ↓
Review
```

## Agent Loop

例如医生上传一批对话：
```
1000个Conversation
```

系统发现：
```
FAQ存在冲突
```

这时候 Orchestrator 可以进入：
```
Conflict
 ↓
Evidence
 ↓
Fact
 ↓
Re-evaluate
```

即：
```
Agent A
 ↓
Agent B
 ↓
发现问题
 ↓
Agent C
 ↓
回到 Agent A
```

## Agent 通信

不要：
```
Agent A
 ↓
一大段文本
 ↓
Agent B
```

应该：
```
Agent A
 ↓
JSON Schema
 ↓
Agent B
```

例如：
```
class FactExtractionResult(BaseModel):

    facts: list[MedicalFact]

    confidence: float

    missing_information: list[str]

    evidence_segments: list[str]
```

利用 Pydantic：
```
result = FactExtractionResult.model_validate(
    llm_output
)
```

LLM 输出不符合 Schema：
```
Validation Error
 ↓
Retry
 ↓
Repair
```

## Prompt 层统一模板
每个 Agent Prompt 都采用：
```
SYSTEM
│
├── Role
├── Objective
├── Input Schema
├── Output Schema
├── Rules
├── Forbidden Behavior
└── Examples

USER
│
└── Dynamic Context
```

例如 Fact Agent：
```
你是一名医疗知识抽取Agent。

任务：
从医生与患者对话中提取医生明确表达的事实。

规则：

1. 只能提取原文明确表达的信息。
2. 不允许根据医学常识补充事实。
3. 不允许改变原始医学含义。
4. 每个Fact必须关联原始证据。
5. 无法确认的内容标记为UNCERTAIN。
6. 不负责给患者提供建议。
```

## Agent Tool设计

| Agent         | 主要 Tool                    |
| ------------- | -------------------------- |
| ASR           | ASR Tool                   |
| Conversation  | Transcript Tool            |
| Topic         | Taxonomy + Embedding       |
| FAQ           | Vector Search + Clustering |
| Fact          | LLM                        |
| Answer        | LLM                        |
| Risk          | Rule Engine + LLM          |
| Evidence      | SQL + Object Storage       |
| Conflict      | Vector Search + LLM        |
| Synthesis     | LLM                        |
| Manual        | RAG + LLM                  |
| Guide Copilot | RAG + Reranker + LLM       |

## Agent 1：ASR Agent

### 职责
负责：
```
音频
 ↓
语音识别
 ↓
说话人识别
 ↓
时间戳
 ↓
分段
```

不是只输出纯文本，而是结构化输出：
```
{
  "conversation_id": "C001",
  "segments": [
    {
      "id": "S001",
      "speaker": "doctor",
      "start": 0.0,
      "end": 5.2,
      "text": "...",
      "confidence": 0.96
    },
    {
      "id": "S002",
      "speaker": "patient",
      "start": 5.5,
      "end": 8.7,
      "text": "...",
      "confidence": 0.93
    }
  ]
}
```

这里一定要保留：
```
音频时间戳 → 文本 → 知识
```

后面医生审核的时候，可以直接点击：
```
“查看原始录音 03:21–03:36”
```

这是非常重要的 Evidence Chain。

### Pipeline
```
Audio
 ↓
Audio Preprocessing
 ↓
VAD
 ↓
ASR
 ↓
Speaker Diarization
 ↓
Punctuation
 ↓
Medical Term Correction
 ↓
Transcript
```

### Tool

抽象成：
```
class ASRTool:

    async def transcribe(
        self,
        audio_path: str
    ):
        pass
```
模型层可以进行替换：
```
Whisper
FunASR
SenseVoice
云端ASR
```
Agent 不直接依赖具体模型。

## Agent 2：Conversation Cleaning Agent

这个 Agent 的作用不是简单“清洗文本”，它真正做的是：

> 恢复对话语义。

例如：
```
患者：
这个做完以后呢？

医生：
这个一般……

患者：
那什么时候能正常？

医生：
差不多一周左右。
```

Agent 应该理解：
```
Topic：
治疗后恢复

Patient Question：
治疗后什么时候能够恢复正常？

Doctor Answer：
通常约一周左右

Question → Answer：
成立
```

输出
```
{
  "turns": [
    {
      "question": "...",
      "answer": "...",
      "topic": "...",
      "doctor_segments": ["S102", "S103"],
      "patient_segments": ["S101"]
    }
  ]
}
```
这个结构非常重要。

因为后面 FAQ Agent 不应该直接处理原始 Transcript，而应该处理：

> Question-Answer Pair

但是注意：
```
不能改变医学含义。
```

所以这里应该是：
```
Original Text
      ↓
Clean Text
      ↓
Semantic Consistency Check
```

## Agent 3：Topic Agent

进行医疗语义分类，不完全使用 LLM 分类。

采用：
```
Embedding
+
Taxonomy
+
LLM
```
### Topic Tree

例如：
```
治疗
├── 治疗方式
├── 治疗周期
├── 治疗效果
├── 治疗疼痛
├── 治疗风险
└── 治疗后恢复

检查
├── 检查必要性
├── 检查流程
├── 检查准备
└── 检查结果

费用
├── 检查费用
├── 治疗费用
└── 复诊费用

术后
├── 饮食
├── 活动
├── 恢复
└── 注意事项
```

输出
```
{
  "topic": "治疗",
  "sub_topic": "治疗后恢复",
  "entities": [
    "治疗项目A"
  ],
  "confidence": 0.91
}
```

如果：

> confidence < 0.7

进入人工确认或者二次 LLM 判断。

## Agent 4：FAQ Agent

知识生产系统中非常核心的 Agent，它解决：

> “不同患者问的其实是不是同一个问题？”


### Pipeline
```
Question
 ↓
Normalize
 ↓
Embedding
 ↓
Vector Search
 ↓
Similarity
 ↓
Cluster
 ↓
LLM Canonical Question
```

例如：
```
“做这个疼吗？”
“治疗会不会疼？”
“治疗疼不疼？”
“会不会特别痛？”
```

Embedding 后：
```
Cluster #102
```

然后 LLM 生成：
```
Canonical Question：

治疗过程中是否会产生疼痛？
```

### FAQ Schema
```
class FAQ(BaseModel):

    faq_id: str

    canonical_question: str

    variants: list[str]

    topic: str

    frequency: int

    confidence: float

    source_conversations: list[str]

    status: str
```

这样最终手册不会有几十个重复问题。

## Agent 5：Medical Fact Agent

不是生成答案，而是：

> 从医生说的话中抽取“事实”。

例如医生说：
```
“一般这种情况治疗以后可能会有一点肿，通常过一两天会慢慢恢复。”
```

提取：
```
{
  "subject": "治疗",
  "type": "treatment_effect",
  "fact": "治疗后可能出现肿胀",
  "duration": "通常1-2天",
  "certainty": "possible",
  "source": "doctor",
  "confidence": 0.92,
  "evidence": [
    {
      "conversation_id": "C001",
      "start": 321.4,
      "end": 329.8
    }
  ]
}
```

### Schema
```
class MedicalFact:

    id: str

    subject: str

    predicate: str

    object: str

    qualifiers: dict

    certainty: str

    scope: str

    source_type: str

    source_id: str

    evidence: list[str]

    status: str
```

最终其实可以形成：
```
Treatment
   │
   ├── may_cause → Swelling
   │
   ├── duration → 1-2 days
   │
   └── recovery → gradual
```

Fact 是整个知识体系的核心对象，为后面的知识图谱提供基础。

### 采用“两阶段抽取”
不要：
```
LLM → Fact
```
而是：
```
第一阶段：
Candidate Fact Extraction

第二阶段：
Fact Verification
```
例如：
```
医生原话
 ↓
Candidate Fact
 ↓
检查是否有原文支持
 ↓
检查是否改变语义
 ↓
Fact
```

这样可以明显降低 LLM 自己“补充医学知识”的风险。

## Agent 6：Answer Agent

负责：

> 把医生的自然语言回答转换成标准医导话术。

输入：
```
FAQ
+
Medical Facts
+
Doctor Original Answer
+
Topic
```

输出：
```
{
  "answer": "...",
  "additional_info": "...",
  "warning": "...",
  "doctor_required": false
}
```

### Prompt 的核心原则

必须明确：
```
你不是医学知识生成器。

只能使用输入提供的事实。

不得补充输入中不存在的医学事实。

不得改变医生表达的医学含义。

如果信息不足，输出 NEED_DOCTOR_REVIEW。
```
这条非常重要。

### 采用“事实约束生成”

不要：
```
LLM自由生成
```

而是：
```
Facts
 ↓
Answer Planning
 ↓
Answer Generation
 ↓
Fact Consistency Check
```

例如：
```
Fact：

可能出现短暂肿胀

↓

Answer：

治疗后部分患者可能会出现短暂肿胀，
具体情况存在个体差异。
```

这里必须区分：
```
医学事实
Knowledge
```

和：
```
对患者的话术
Communication Template
```

两者不能混在一起。

## Agent 7：Risk Agent

是：
> LLM + Rule Engine 双引擎。

它负责识别：
```
高风险回答
禁止承诺
可能误导
需要医生判断
可能涉及诊断
可能涉及治疗方案
可能涉及药物剂量
可能涉及紧急情况
```

例如 Agent生成：
```
“这种情况不用去医院。”
```

Risk Agent 应该直接标记：
```
RISK_HIGH
```

并阻止进入：
```
Approved Guide Knowledge
```

Rule Engine

例如：

HIGH_RISK_RULES = [
    "diagnosis",
    "drug_dosage",
    "treatment_change",
    "emergency",
    "absolute_guarantee"
]

### 规则

例如：
```
如果回答：

“你肯定没事”
→ HIGH

“你一定是XXX”
→ HIGH

“药物可以增加到……”
→ HIGH

“可以停药”
→ HIGH
```
这里不要依赖 LLM 判断。

### Risk Agent 输出
```
{
  "risk_level": "HIGH",
  "risk_types": [
    "diagnosis",
    "treatment_advice"
  ],
  "reason": "...",
  "action": "DOCTOR_REVIEW"
}
```

## Agent 8：Evidence Agent

负责回答：
```
“这句话到底是从哪里来的？”
```

是：

> Evidence Retrieval Service + Agent。

负责建立：
```
Knowledge
 ↓
Evidence
 ↓
Original Conversation
 ↓
Audio Timestamp
```

例如
```
{
  "knowledge_id": "K001",
  "evidence": [
    {
      "conversation_id": "C001",
      "segment_id": "S102",
      "start": 321.2,
      "end": 338.4,
      "text": "……"
    }
  ]
}
```

医生审核时：
```
AI知识
  ↓
查看原文
  ↓
播放原音频
```
这样系统的可解释性会非常强。


最终形成：
```
Knowledge
   │
   ├── Source Conversation
   ├── Audio Timestamp
   ├── Original Text
   ├── Doctor
   └── Created Time
```

这样整个知识库是可追溯的。

## Agent 9：Conflict Agent

解决：

> 不同时间、不同医生、不同患者情况下的知识冲突。

这个非常重要。

假设：
```
医生3月份说：

“一般建议治疗后休息3天。”

医生8月份又说：

“现在一般休息1天就可以。”
```

不能自动：
```
DELETE OLD
```

而应该：
```
Knowledge Conflict
        ↓
Doctor Review
```

应该产生：
```
Knowledge Conflict

旧版本：
休息3天

新版本：
休息1天

来源：
2026-03
2026-08

状态：
REQUIRES_REVIEW
```

然后交给医生确认。

### Pipeline：
```
New Fact
 ↓
Vector Search Existing Facts
 ↓
Same Entity?
 ↓
Same Predicate?
 ↓
Conflict Detection
 ↓
LLM Reasoning
 ↓
Conflict Record
```

### 判断模型

可以把知识关系定义成：
```
SUPPORT
CONTRADICT
REFINE
DUPLICATE
CONTEXT_DIFFERENT
```
例如：
```
Fact A：
通常需要休息3天

Fact B：
某类患者通常休息1天
```
不是 Conflict。

而是：
```
Fact B
  └── REFINE
       └── Fact A
```

所以这里不能简单字符串比较。

## Agent 10：Knowledge Synthesis Agent

负责把前面所有结果组合成最终知识卡片，是：
> 组装知识

输入：
```
FAQ
+
Facts
+
Answer
+
Risk
+
Evidence
+
Conflict
```

输出：
```
Knowledge Card
```

### Knowledge Card

```
{
  "knowledge_id": "K001",

  "question": "治疗后多久可以恢复？",

  "topic": "治疗/恢复",

  "answer": "...",

  "facts": [
    "..."
  ],

  "scope": "...",

  "warnings": [
    "..."
  ],

  "doctor_required": false,

  "risk_level": "LOW",

  "evidence": [
    "C001:S102"
  ],

  "version": 3,

  "status": "PENDING_REVIEW"
}
```

## Agent 11：Manual Generation Agent

不是直接读取原始对话，而是：
```
Approved Knowledge
        ↓
Knowledge Grouping
        ↓
Chapter Planning
        ↓
Section Generation
        ↓
Consistency Check
        ↓
Manual
```

例如：
```
第一章 常见问题

1. 治疗前
2. 治疗中
3. 治疗后

第二章 高频问题

2.1 疼痛
2.2 恢复
2.3 饮食
2.4 活动

第三章 特殊情况

3.1 出现异常症状
3.2 患者要求修改治疗方案
3.3 患者询问药物
```

## Agent 12：Guide Copilot Agent

最终真正给医导使用的 Agent。

医导输入：
```
“患者问做完治疗之后能不能马上运动？”
```

Agent：
```
                         User Question
                               ↓
                       Input Guardrail
                               ↓
                       Intent Agent
                               ↓
                        Query Rewrite
                               ↓
                     Knowledge Retrieval
                               ↓
                        Reranker
                               ↓
                    Approved Knowledge
                               ↓
                       Risk Detection
                               ↓
                      Answer Generation
                               ↓
                       Output Guardrail
                               ↓
                         Guide Answer
```

返回：
```
建议回答：

“具体恢复时间需要结合您的实际情况判断。
如果是刚完成治疗，建议先按照医生给出的恢复要求进行。
如果您告诉我具体进行了哪种治疗，我可以帮您查询对应的注意事项。”

⚠️ 医导注意：
不要自行判断患者是否可以运动。
```

这比让医导直接问 ChatGPT 安全很多。


### 核心原则

必须：

> Approved Knowledge Only

即：
```
documents = vector_db.search(query)

documents = [
    x for x in documents
    if x.status == "APPROVED"
]
```
如果找不到：
```
没有经过医生审核的相关知识，
请将该问题转交医生。
```
而不是让 LLM 自己发挥。

# Agent Flow 设计

两个核心 Flow + 一个审核 Flow

## Flow A：医生对话 → 知识
```
                    ┌──────────────┐
                    │   Audio      │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │   ASR Agent  │
                    └──────┬───────┘
                           ↓
                ┌────────────────────┐
                │ Conversation Agent │
                └─────────┬──────────┘
                          ↓
                ┌────────────────────┐
                │    Topic Agent     │
                └─────────┬──────────┘
                          ↓
             ┌────────────┴─────────────┐
             ↓                          ↓
       ┌───────────┐              ┌─────────────┐
       │ FAQ Agent │              │ Fact Agent  │
       └─────┬─────┘              └──────┬──────┘
             │                           │
             └────────────┬──────────────┘
                          ↓
                  ┌──────────────┐
                  │ Answer Agent │
                  └──────┬───────┘
                         ↓
                  ┌──────────────┐
                  │  Risk Agent  │
                  └──────┬───────┘
                         ↓
                ┌─────────────────┐
                │ Evidence Agent  │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │ Conflict Agent  │
                └────────┬────────┘
                         ↓
              ┌─────────────────────┐
              │ Knowledge Synthesis │
              └──────────┬──────────┘
                         ↓
                   Doctor Review
```

## Flow B：医生审核
```
AI生成知识
     ↓
风险检查
     ↓
医生审核
     │
     ├── Reject
     │
     ├── Modify
     │
     └── Approve
             ↓
       Knowledge Version
             ↓
       Published KB
```

医生看到的不是一大篇文章。

而是：

FAQ

患者问题
```
做完治疗多久可以恢复？
```
AI提炼答案
```
一般情况下……
```
原始医生回答
```
……
```
来源
```
对话 #1023
12:31–13:02
```
AI风险
```
中风险：涉及恢复周期
```
医生操作
```
[通过]

[修改]

[拒绝]

[查看原音频]
```
这样医生审核效率会非常高。

## Flow C：医导实际使用
```
患者问题
     ↓
Guide Copilot
     ↓
Intent Agent
     ↓
Query Rewrite
     ↓
RAG Retrieval
     ↓
Knowledge Filter
     ↓
Evidence Check
     ↓
Risk Agent
     ↓
Answer Agent
     ↓
医导答案
```

特别重要的一点：

> 只能从 Approved Knowledge 中回答。

而不是：

> Internet + LLM

否则医导系统会变成一个不可控的医疗聊天机器人。

# 知识库设计
建立结构化 Knowledge Model。
```
Knowledge
│
├── FAQ
│
├── MedicalFact
│
├── AnswerTemplate
│
├── RiskRule
│
├── Entity
│
├── Evidence
│
├── Conversation
│
└── KnowledgeVersion
```

## FAQ
```
{
  "id": "FAQ001",
  "question": "治疗后多久可以恢复？",
  "variants": [
    "多久能恢复？",
    "治疗后多久正常？"
  ],
  "topic": "恢复",
  "status": "approved"
}
```
## AnswerTemplate
```
{
  "faq_id": "FAQ001",
  "answer": "...",
  "scope": "...",
  "warning": "...",
  "status": "approved"
}
```
## Evidence
```
{
  "knowledge_id": "K001",
  "conversation_id": "C102",
  "audio_start": 321.2,
  "audio_end": 338.7,
  "speaker": "doctor"
}
```

# 知识状态机
```
EXTRACTED
    ↓
STRUCTURED
    ↓
RISK_CHECKED
    ↓
PENDING_REVIEW
    │
    ├─────────────┐
    ↓             ↓
REJECTED      MODIFIED
                  ↓
              APPROVED
                  ↓
              PUBLISHED
                  ↓
              DEPRECATED
```
这样就不会出现：

> AI今天生成了一句话，明天医导就拿去给患者用了。


# 知识生命周期
```
医生对话
   ↓
知识发现
   ↓
知识抽取
   ↓
知识验证
   ↓
医生审核
   ↓
知识发布
   ↓
医导使用
   ↓
患者反馈
   ↓
低置信度问题
   ↓
重新进入知识生产
```


# 知识自动发现
系统发现：
```
过去一个月：

患者问题：
“治疗后疼不疼？”

出现 132 次
```
但是知识库只有：
```
“治疗后恢复时间”
```
系统自动产生：
```
Knowledge Gap

Topic:
治疗体验

Candidate FAQ:
治疗过程中会不会疼？

Frequency:
132

Priority:
High

Evidence:
37段医生对话
```
然后通知医生：

> 发现一个高频但当前知识库缺失的问题，是否生成标准答案？

医生使用AI自动生成草稿。

# 医生个人知识画像

随着时间积累：
```
医生
 │
 ├── 高频问题
 ├── 常用解释方式
 ├── 专业术语
 ├── 常见患者群体
 ├── 常见风险
 ├── 典型回答
 └── 未覆盖问题
```

变成：**医生个人知识资产数字化系统**

# 注意点

## Kafka
为了未来可以扩展，使用Kafka作为中间件，而不用重构整个系统。

## Model Gateway
不是所有任务都调用最大的模型
```
                    LLM Gateway
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Qwen          GPT/其他        小模型
          │
          ↓
     Agent Routing
```

| 任务                | 模型              |
| ----------------- | --------------- |
| 文本清洗              | 小模型             |
| Topic分类           | 小模型             |
| FAQ聚类             | Embedding + 小模型 |
| Fact Extraction   | 强模型             |
| Answer Generation | 强模型             |
| Risk Detection    | 强模型 + Rule      |
| 手册生成              | 强模型             |
| RAG回答             | 强模型             |

## Rule Engine
作为医疗类型的Agent，为了专业严谨性，需要有规则约束，禁止LLM随意回答。
```
LLM
 +
Medical Rule Engine
 +
Knowledge Permission
 +
Doctor Approval
```

形成四重约束

比如：
```
如果：

涉及药物剂量
OR
涉及诊断
OR
涉及治疗方案修改
OR
涉及急症
OR
知识没有医生审核

→ 禁止自动回答
→ 转人工/医生
```

## Agent 的安全护栏

```
                User Query
                    ↓
                Input Guardrail
                    ↓
                Intent Detection
                    ↓
                Knowledge RAG
                    ↓
                Answer Agent
                    ↓
                Risk Agent
                    ↓
            Output Guardrail
                    ↓
                Final Answer
```
其中：

### Input Guardrail

识别：
```
普通咨询
医疗咨询
紧急情况
诊断请求
处方请求
治疗调整
```

### Output Guardrail

检查：
```
是否超出知识库
是否出现幻觉
是否给出未经批准建议
是否存在绝对化承诺
是否涉及诊断
是否涉及药物调整
```

# 产品端
分为三端：医生端、医导端和管理端

## 医生端
```
录音
 ↓
知识发现
 ↓
审核
 ↓
修改
 ↓
发布
```

## 医导端
```
患者问题
 ↓
AI推荐回答
 ↓
标准话术
 ↓
注意事项
 ↓
需要医生介入？
```

## 管理端
```
知识库
FAQ统计
知识覆盖率
知识版本
审核记录
风险记录
Agent运行记录
```
