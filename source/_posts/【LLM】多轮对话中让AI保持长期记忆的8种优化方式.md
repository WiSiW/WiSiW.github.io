---
title: 【LLM】多轮对话中让AI保持长期记忆的8种优化方式
date: 2026-11-03 15:38:14
tags:
    - 多轮对话
    - 长期记忆
    - LangChain
---

# 【LLM】多轮对话中让AI保持长期记忆的8种优化方式

## 一、获取全量历史对话

### 1.1 核心思想是
将用户与 AI 的所有历史消息完整保存，在每次请求时，将历史消息全部加载到 Prompt 中，让大模型根据完整上下文回答。

### 1.2 具体代码实现
```python
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.prebuilt import ToolNode
from langchain_core.messages import SystemMessage

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)

def chat_node(state: MessagesState):
    messages = [
        SystemMessage(content="你是一个能够记住历史对话的AI助手。"),
        *state["messages"]
    ]

    response = llm.invoke(messages)

    return {"messages": [response]}


builder = StateGraph(MessagesState)
builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")

# 创建 Checkpointer
checkpointer = InMemorySaver()

# 编译
graph = builder.compile(checkpointer=checkpointer)


def chat(user_id, thread_id, question):
    config = {
        "configurable": {
            "thread_id": thread_id
        }
    }

    result = graph.invoke(
        {
            "messages": [
                HumanMessage(content=question)
            ]
        },
        config=config
    )

    return result["messages"][-1].content


if __name__ == "__main__":
    print(chat("user_001", "session_001",
               "我正在开发一个知识中台"))

    print(chat("user_001", "session_001",
               "后端使用SpringBoot"))

    print(chat("user_001", "session_001",
               "我的后端使用什么技术？"))
```


优缺点：
- 优点：实现简单、信息完整、不需要额外的摘要或信息提取模型。
- 缺点：Token 消耗随着历史对话增长而增长，容易超出上下文窗口，同时可能引入大量无关信息。

适用场景： 短期客服、简单问答、需要完整上下文的小规模对话。

## 二、滑动窗口获取最近部分对话内容

### 2.1 核心思想
只保留最近 K 轮对话，超过窗口范围的历史消息自动丢弃。

### 2.2 具体代码实现
```python
from langchain_ollama import ChatOllama
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    SystemMessage
)

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)
def get_recent_messages(messages, k=2):
    """
    保留最近 k 轮完整对话。
    一轮 = 一条 HumanMessage + 一条 AIMessage。
    """

    # 过滤系统消息
    conversation = [
        msg for msg in messages
        if isinstance(msg, (HumanMessage, AIMessage))
    ]

    # 每轮通常包含两条消息
    recent = conversation[-(k * 2):]

    # 确保从用户消息开始，避免留下不完整对话
    if recent and isinstance(recent[0], AIMessage):
        recent = recent[1:]

    return recent


def chat_with_window(messages, question, k=2):
    recent_messages = get_recent_messages(messages, k)

    prompt = [
        SystemMessage(
            content="你是一个电商客服助手，请结合最近对话回答。"
        ),
        *recent_messages,
        HumanMessage(content=question)
    ]

    return llm.invoke(prompt).content
```

### 2.3 基于Token数的窗口优化
根据 Token 数量裁剪

```python
from langchain_core.messages import trim_messages

def trim_history(messages, llm, max_tokens=4000):
    return trim_messages(
        messages,
        max_tokens=max_tokens,
        token_counter=llm,
        strategy="last",
        include_system=True,
        start_on="human",
        allow_partial=False
    )
```

优缺点：
- 优点：响应速度快，Token 消耗可控，实现简单。
- 缺点：早期对话中的重要信息可能被丢弃，无法解决跨会话记忆。

适用场景： 电商咨询、实时客服、短期连续问答。


## 三、获取历史对话中实体信息

### 3.1 核心思想
与保存整段对话不同，实体记忆主要关注历史对话中出现的关键实体，以及这些实体对应的属性和描述。

### 3.2 具体代码实现

#### 定义实体结构
```python
# entityExtraction.py
from pydantic import BaseModel, Field
from typing import List, Optional


class Entity(BaseModel):
    name: str = Field(description="实体名称")
    entity_type: str = Field(
        description="实体类型，例如人物、组织、项目、技术、产品"
    )
    description: str = Field(
        description="根据历史对话提取的实体描述"
    )
    attributes: dict = Field(
        default_factory=dict,
        description="实体相关属性"
    )
    confidence: float = Field(
        default=1.0,
        description="实体提取置信度"
    )


class EntityExtraction(BaseModel):
    entities: List[Entity] = Field(
        description="从历史对话中提取的实体列表"
    )
```

#### 使用 LLM 提取实体
```python
# app.py
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage, AIMessage
from langchain_core.prompts import ChatPromptTemplate
from entityExtraction import EntityExtraction

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0.7,
    base_url="http://127.0.0.1:11434"
)

# 使用结构化输出
extractor = llm.with_structured_output(
    EntityExtraction
)

prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """
        你是一个对话实体信息提取专家。

        请从历史对话中提取所有有价值的实体。

        要求：
        1. 提取人物、组织、项目、技术、产品等实体。
        2. 提取实体的相关属性。
        3. 合并同一实体在不同对话中的信息。
        4. 不要凭空推测不存在的信息。
        5. 对于信息不明确的属性，不要编造。
        6. 保留实体之间有明确依据的关联。
        """
    ),
    (
        "human",
        "历史对话：\n{history}"
    )
])

chain = prompt | extractor


def extract_entities(messages):
    history = "\n".join([
        f"{'用户' if isinstance(m, HumanMessage) else 'AI'}：{m.content}"
        for m in messages
        if isinstance(m, (HumanMessage, AIMessage))
        and isinstance(m.content, str)
    ])

    result = chain.invoke({
        "history": history
    })

    return result


# 模拟历史对话
messages = [
    HumanMessage(content="我正在开发一个知识中台"),
    AIMessage(content="你使用什么技术开发？"),
    HumanMessage(content="后端使用SpringBoot"),
    HumanMessage(content="文档检索使用Elasticsearch"),
    HumanMessage(content="缓存使用Redis")
]

result = extract_entities(messages)

for entity in result.entities:
    print(entity.model_dump())
```

输出：
```
{'name': '知识中台', 'entity_type': '项目', 'description': '正在开发的知识管理系统', 'attributes': {'开发状态': '开发中', '主要功能': '知识管理'}, 'confidence': 0.95}
{'name': 'SpringBoot', 'entity_type': '技术', 'description': '后端开发框架', 'attributes': {'技术类型': '后端框架', '应用场景': '后端开发'}, 'confidence': 0.95}
{'name': 'Elasticsearch', 'entity_type': '技术', 'description': '分布式搜索引擎', 'attributes': {'技术类型': '搜索引擎', '应用场景': '文档检索'}, 'confidence': 0.95}
{'name': 'Redis', 'entity_type': '技术', 'description': '内存数据库', 'attributes': {'技术类型': '内存数据库', '应用场景': '缓存'}, 'confidence': 0.95}
```

优缺点：
- 优点：能够把非结构化对话转化为结构化知识，方便查询和更新。
- 缺点：依赖实体抽取准确率，实体消歧和属性冲突处理比较复杂。

适用场景： 用户画像、项目助手、企业知识助手、长期个人助理。

## 四、利用知识图谱获取历史对话中的实体及其联系

### 4.1 核心原理
实体记忆主要关注实体本身，而知识图谱进一步关注实体之间的关系。

### 4.2 具体实现

#### 定义图谱结构
```python
# graph_extractor.py
from typing import Any, Optional
from pydantic import BaseModel, Field


class Entity(BaseModel):
    name: str = Field(description="实体名称")

    entity_type: str = Field(
        description="实体类型：人物、组织、项目、技术、产品等"
    )

    description: str = Field(
        description="实体描述"
    )

    attributes: dict[str, Any] = Field(
        default_factory=dict
    )

    confidence: float = Field(
        default=1.0,
        ge=0,
        le=1
    )

class Relation(BaseModel):
    source: str = Field(description="关系起点实体")
    target: str = Field(description="关系终点实体")
    relation: str = Field(description="关系类型")
    evidence: str = Field(description="原始对话证据")
    confidence: float = Field(default=1.0)


class KnowledgeGraph(BaseModel):
    entities: list[Entity] = Field(default_factory=list)
    relations: list[Relation] = Field(default_factory=list)
```

#### 使用 LLM 提取关系
```python
# app.py
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from graph_extractor import KnowledgeGraph

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)

graph_extractor = llm.with_structured_output(
    KnowledgeGraph
)

prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """
        你是知识图谱构建专家。

        从对话中提取实体及其关系。

        关系必须满足：
        1. 必须有原文依据。
        2. 不允许推测隐含关系。
        3. 实体名称必须统一。
        4. 每个关系必须包含证据。
        5. 不确定的关系不要提取。
        """
    ),
    ("human", "{history}")
])

chain = prompt | graph_extractor


result = chain.invoke({
    "history": """
    小李是一名程序员。
    莫尔索是小李的笔名。
    小李正在开发知识中台。
    知识中台后端使用SpringBoot。
    """
})

for relation in result.relations:
    print(relation.model_dump())
```

输出：
```
{'source': '小李', 'target': '程序员', 'relation': '职业', 'evidence': '小李是一名程序员。', 'confidence': 1.0}
{'source': '莫尔索', 'target': '小李', 'relation': '笔名', 'evidence': '莫尔索是小李的笔名。', 'confidence': 1.0}
{'source': '小李', 'target': '知识中台', 'relation': '开发', 'evidence': '小李正在开发知识中台。', 'confidence': 1.0}
{'source': '知识中台', 'target': 'SpringBoot', 'relation': '使用', 'evidence': '知识中台后端使用SpringBoot。', 'confidence': 1.0}
```

优缺点：
- 优点：擅长复杂关联查询、多跳推理、实体关系追踪。
- 缺点：建设成本较高，实体消歧、图谱更新和关系质量控制比较复杂。

适用场景： 企业知识助手、复杂业务 Agent、医疗知识关联、人物关系分析、项目管理助手。


## 五、对历史对话进行阶段性总结摘要

### 5.1 核心思想
不再保存所有历史对话的详细内容，而是利用 LLM 对历史对话进行总结，生成一份简洁的上下文摘要。

实现方案
设计三个核心模块：
- 1. 历史消息收集：收集需要总结的历史对话。
- 2. 摘要生成：LLM 提取用户背景、项目进展、重要决策。
- 3. 摘要更新：将新摘要与旧摘要合并，避免覆盖之前的重要信息。

### 5.2 具体代码实现
```python
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)

summary_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """
        你是一个对话长期记忆摘要专家。

        请根据历史对话生成长期记忆摘要。

        重点保留：
        1. 用户背景和专业技能。
        2. 用户长期目标。
        3. 正在进行的项目。
        4. 已确定的技术方案。
        5. 重要决策和待解决问题。

        不要保留无意义的闲聊。
        不要编造信息。
        对存在冲突的信息保留时间和状态。
        输出简洁、结构化的摘要。
        """
    ),
    (
        "human",
        """
        原有摘要：
        {old_summary}

        新增对话：
        {new_history}

        请生成更新后的完整摘要。
        """
    )
])

summary_chain = summary_prompt | llm


def update_summary(old_summary, new_history):
    result = summary_chain.invoke({
        "old_summary": old_summary,
        "new_history": new_history
    })

    return result.content


old_summary = ""

new_history = """
用户正在开发知识中台。
后端使用SpringBoot。
文档检索使用Elasticsearch。
缓存使用Redis。
"""

new_summary = update_summary(
    old_summary,
    new_history
)

print(new_summary)
```
输出：
```
**长期记忆摘要**  
1. **用户背景与专业技能**  
   - 具备后端开发经验，熟悉Java生态技术栈。  

2. **长期目标**  
   - 构建高效、可扩展的知识中台，支持企业级文档管理与智能检索。  

3. **正在进行的项目**  
   - 知识中台开发，聚焦文档存储、检索与缓存优化。  

4. **已确定的技术方案**  
   - 后端框架：SpringBoot（Java）  
   - 文档检索：Elasticsearch（支持全文搜索与结构化查询）  
   - 缓存方案：Redis（用于加速高频数据访问）  

5. **重要决策与待解决问题**  
   - 技术选型已确认，需解决多系统集成（SpringBoot + Elasticsearch + Redis）的兼容性问题。  
   - 待定：数据一致性保障机制、缓存更新策略及搜索性能优化方案。
```

优缺点：
- 优点：大幅降低上下文长度，能够长期保留重要信息。
- 缺点：摘要具有信息损失，容易遗漏数字、具体参数、否定条件等细节。

适用场景： 教育辅导、长期技术支持、个人助理、长时间任务型 Agent。


## 六、需要获取最新对话，又要兼顾较早历史对话

### 6.1 核心原理
结合了：全量历史摘要 与 最近 N 轮原始消息。

与单纯摘要相比，它能够兼顾长期信息和近期对话细节。
```
历史对话（100轮）
       |
       v
历史摘要
       |
       +----------------+
       |                |
       v                v
  长期项目背景       最近10轮原始消息
       |                |
       +--------+-------+
                |
                v
             LLM回答
```

### 6.2 具体代码实现
```python
from langchain_ollama import ChatOllama
from langchain_core.messages import (
    SystemMessage,
    HumanMessage,
    AIMessage
)

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)

def build_summary_buffer(
    summary,
    recent_messages,
    question
):
    prompt = [
        SystemMessage(
            content=f"""
            你是一个长期记忆AI助手。

            以下是较早历史对话的摘要：
            {summary}

            请结合最近对话回答问题。
            如果摘要和最近对话存在冲突，
            优先参考有明确时间依据的最新事实。
            """
        ),
        *recent_messages,
        HumanMessage(content=question)
    ]

    return llm.invoke(prompt).content
```

优缺点：
- 优点：同时保留长期背景和短期细节，Token 消耗相对可控。
- 缺点：需要设计摘要更新策略，避免摘要遗漏重要信息。

适用场景： 长期技术支持、复杂项目 Agent、多阶段任务助手。


## 七、回溯最近和最关键的对话信息

### 7.1 核心思想
```
历史消息
   |
   v
Token统计
   |
   v
是否超过4000 Token？
   |
   +-- 否 --> 保留所有消息
   |
   +-- 是 --> 从最早消息开始裁剪
                 |
                 v
             保留最近消息
```

### 7.2 具体代码实现
```python
from langchain_core.messages import trim_messages

def get_token_limited_history(
    messages,
    llm,
    max_tokens=4000
):
    return trim_messages(
        messages,
        token_counter=llm,
        max_tokens=max_tokens,
        strategy="last",
        include_system=True,
        start_on="human",
        allow_partial=False
    )
```

优缺点：
- 优点：精确控制上下文长度，适应不同模型的 Token 限制。
- 缺点：不具备语义重要性判断，较早但关键的信息仍可能被裁剪。

适用场景： 长文本问答、代码助手、长上下文 Agent、模型调用成本控制。


## 八、基于向量检索对话信息

### 8.1 核心思想
将历史对话切分成独立的记忆片段，通过 Embedding 模型将文本转换成向量，保存到向量数据库中。

```
当前问题
   |
   v
Embedding模型
   |
   v
问题向量
   |
   v
向量数据库
   |
   v
相似度检索 TopK
   |
   v
历史记忆：我平时喜欢跑步
   |
   v
LLM生成答案
```

### 8.2 具体代码实现
```python
# vector_memory.py
from langchain_chroma import Chroma
from langchain_ollama import OllamaEmbeddings
from langchain_core.documents import Document

embeddings = OllamaEmbeddings(
    model="nomic-embed-text",
    base_url="http://127.0.0.1:11434"
)

vectorstore = Chroma(
    collection_name="conversation_memory",
    embedding_function=embeddings,
    persist_directory="./chroma_db"
)
```

#### 保存历史对话
```python
def save_memory(user_id, content, category="general"):
    document = Document(
        page_content=content,
        metadata={
            "user_id": user_id,
            "category": category
        }
    )

    vectorstore.add_documents([document])


save_memory(
    "user_001",
    "我喜欢吃火锅",
    "preference"
)

save_memory(
    "user_001",
    "我平时喜欢跑步",
    "hobby"
)

save_memory(
    "user_001",
    "我正在学习LangChain",
    "learning"
)
```

#### 根据问题检索历史记忆
```python
def retrieve_memory(user_id, query, k=3):
    docs = vectorstore.similarity_search(
        query,
        k=k,
        filter={
            "user_id": user_id
        }
    )

    return [
        doc.page_content
        for doc in docs
    ]


memories = retrieve_memory(
    "user_001",
    "我喜欢什么运动？",
    k=3
)

print(memories)
```

输出：
```
['我平时喜欢跑步', '我喜欢吃火锅', '我正在学习LangChain']
```

#### 将检索结果交给 LLM
```python
from langchain_ollama import ChatOllama
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0,
    base_url="http://127.0.0.1:11434"
)
prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """
        你是一个具有长期记忆能力的AI助手。

        以下是从历史对话中检索出的相关记忆：
        {memories}

        回答要求：
        1. 优先根据历史记忆回答。
        2. 不允许编造用户没有表达过的信息。
        3. 如果没有找到相关记忆，明确说明不知道。
        """
    ),
    ("human", "{question}")
])


def chat_with_memory(user_id, question):
    memories = retrieve_memory(
        user_id,
        question
    )

    chain = prompt | llm

    result = chain.invoke({
        "memories": "\n".join(memories),
        "question": question
    })

    return result.content

res = chat_with_memory(
    "user_001",
    "我喜欢什么运动？"
)

print(res)
```

输出：
```
根据你的描述，你平时喜欢跑步。这是目前唯一提到的运动爱好。
```

优缺点：
- 优点：适合海量历史记忆，支持语义检索，能够找回很久以前的相关信息。
- 缺点：向量相似度不等于事实正确性，存在召回不准确、信息冲突和语义混淆问题。

适用场景： 个人 AI 助手、长期客服、知识问答、跨会话 Agent。

## 九、八种记忆方式的对比与选型

### 9.1 对比
记忆方式 | 主要解决的问题 | 是否适合跨会话 | 推荐程度
---|---|---|---
全量历史对话 | 保持完整上下文 | 需要持久化支持 | 高
滑动窗口 | 控制最近对话长度 | 否 | 高
实体记忆 | 保存人物、项目、技术等事实 | 是 | 很高
知识图谱 | 保存实体之间的关系 | 是 | 按业务需要
对话摘要 | 压缩长期历史 | 是 | 很高
摘要 + 最近消息 | 兼顾历史和当前上下文 | 是 | 很高
Token Buffer | 控制上下文 Token | 否 | 高
向量检索 | 找回语义相关的历史信息 | 是 | 很高

### 9.2 选型
业务场景 | 推荐组合
---|---
普通客服机器人 | 全量历史 + 滑动窗口
长期个人 AI 助手 | 摘要 + 用户画像 + 向量检索
企业知识助手 | 实体记忆 + 向量检索 + 知识图谱
长期项目开发 Agent | 摘要 + 实体记忆 + 项目专题记忆
医疗咨询助手 | 实体记忆 + 知识图谱 + 证据溯源
复杂任务型 Agent | Checkpointer + 摘要 + 长期记忆 Store