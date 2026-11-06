---
title: 【LangGraph】基于 LangGraph 构建完整的长期记忆系统
date: 2026-11-04 15:38:14
tags:
    - 多轮对话
    - 长期记忆
    - LangGraph
---

# 【LangGraph】基于 LangGraph 构建完整的长期记忆系统

## 一、整体架构设计
整个系统可以划分为五个核心模块。

### 模块一：对话管理模块

负责接收用户输入、维护会话状态。

主要功能：
- 创建和管理 Conversation。
- 通过 thread_id 管理会话。
- 保存完整历史消息。
- 管理最近消息窗口。
- 控制上下文 Token 数量。
推荐使用 LangGraph Checkpointer。

### 模块二：记忆提取模块

负责从历史对话中识别具有长期价值的信息。

主要功能：
- 实体提取。
- 用户偏好提取。
- 项目信息提取。
- 事实和关系抽取。
- 对话摘要生成。
- 记忆重要性评分。
推荐使用 LLM Structured Output。

### 模块三：记忆存储模块

负责对不同类型的记忆进行分类存储。
存储介质 | 存储内容
---|---
PostgreSQL / MySQL | 用户画像、实体、结构化事实、记忆元数据
Redis | 热门记忆缓存、会话状态缓存
Chroma / Milvus | 向量化历史记忆
Neo4j | 实体及其关系
PostgreSQL Checkpointer | 会话状态及消息历史

### 模块四：记忆检索模块

根据用户当前问题，动态选择合适的记忆。

例如：
```
用户：我之前开发的知识中台采用了什么架构？

          |
          v
       Query分析
          |
          v
       记忆路由
          |
     +----+-----+------+
     |          |      |
     v          v      v
  实体检索    向量检索  摘要检索
     |          |      |
     +----------+------+
                |
                v
             结果合并
                |
                v
             Reranker
                |
                v
           构建记忆上下文
```

### 模块五：记忆管理模块

负责记忆的生命周期管理：
- 记忆新增。
- 记忆更新。
- 记忆去重。
- 记忆冲突处理。
- 记忆过期。
- 记忆删除。
- 记忆权限控制。

## 二、使用 LangGraph Store 实现跨会话长期记忆

区分：
- Checkpointer：保存当前 Thread 的短期记忆。
- Store：保存跨 Thread 的长期记忆。

### 2.1 核心原理
用户开启了三个会话：
```
用户 user_001
    |
    +-- thread_001：讨论知识中台
    |
    +-- thread_002：讨论智能问数
    |
    +-- thread_003：讨论Agent开发
```

三个会话各自有独立的 Checkpointer。

但是它们都可以访问：
```
长期记忆 Store
    |
    +-- 用户画像
    +-- 技术偏好
    +-- 项目知识
    +-- 历史事实
```

### 2.2 创建长期记忆 Store
```python
from langgraph.store.memory import InMemoryStore

store = InMemoryStore(
   index={
      "dims": 768,
      "embed": embeddings
   }
)

user_id = "user_001"

namespace = (
   "users",
   user_id,
   "memories"
)
```

### 2.3 保存长期记忆
```python
def save_long_term_memory(
   store,
   user_id,
   memory_id,
   content,
   category
):
   namespace = (
      "users",
      user_id,
      "memories"
   )

   store.put(
      namespace,
      memory_id,
      {
         "content": content,
         "category": category
      }
   )


save_long_term_memory(
   store,
   "user_001",
   "memory_001",
   "用户正在开发知识中台，后端使用SpringBoot",
   "project"
)

save_long_term_memory(
   store,
   "user_001",
   "memory_002",
   "用户喜欢使用Python开发AI Agent",
   "preference"
)
```

### 2.4 根据当前问题检索长期记忆
```python
def search_long_term_memory(
   store,
   user_id,
   query,
   limit=5
):
   namespace = (
      "users",
      user_id,
      "memories"
   )

   results = store.search(
      namespace,
      query=query,
      limit=limit
   )

   return [
      item.value["content"]
      for item in results
   ]


memories = search_long_term_memory(
   store,
   "user_001",
   "用户之前开发过什么项目？"
)

print(memories)
```

输出：
```
['用户正在开发知识中台，后端使用SpringBoot', '用户喜欢使用Python开发AI Agent']
```

### 2.5 让 Agent 自动读取长期记忆
将长期记忆检索封装成 Tool：
```python
from langchain.tools import tool

@tool
def search_user_memory(query: str) -> str:
   """
   查询用户过去的长期记忆，
   包括用户偏好、项目背景、技术选型等。
   """

   memories = search_long_term_memory(
      store,
      user_id,
      query
   )

   return "\n".join(memories) if memories else "没有找到相关记忆"
```

将这个 Tool 注册到 Agent 中：
```python
from langchain_ollama import ChatOllama
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver

llm = ChatOllama(
   model="qwen3:8b",
   temperature=0,
   base_url="http://127.0.0.1:11434"
)
checkpointer = InMemorySaver()

agent = create_agent(
   model=llm,
   tools=[search_user_memory],
   system_prompt="""
   你是一个具有长期记忆能力的AI助手。

   当用户的问题涉及以前的项目、技术偏好、
   历史事实时，优先调用 search_user_memory。

   必须以检索到的事实为依据回答，
   不允许编造历史信息。
   """,
   checkpointer=checkpointer,
   store=store
)
```

```python
config = {
   "configurable": {
      "thread_id": "thread_003"
   }
}

result = agent.invoke(
   {
      "messages": [
         {
            "role": "user",
            "content": "我之前开发过什么项目？"
         }
      ]
   },
   config=config
)

print(result["messages"][-1].content)
```

输出：
```
根据您的历史记录：

1. **知识中台项目**（当前进行中）
   - 技术栈：SpringBoot（后端框架）
   - 项目方向：知识整合与知识图谱相关功能开发

2. **AI Agent开发偏好**
   - 首选语言：Python
   - 应用场景：AI智能体开发（具体领域未明确，但体现对AI工程化实践的兴趣）

是否需要进一步了解某个项目的具体技术细节或开发进度？
```