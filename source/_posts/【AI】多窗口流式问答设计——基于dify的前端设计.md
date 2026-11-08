---
title: 【AI】多窗口流式问答设计（前后端）
date: 2026-09-16 15:38:14
tags:
    - 多窗口问答
    - 单tab
    - dify
---

# 【AI】多窗口流式问答设计（基于dify的前端设计）

## 需求

同一个标签页多个独立聊天会话

```
窗口 A → conversation_A → Dify
窗口 B → conversation_B → Dify
窗口 C → conversation_C → Dify
```

核心思想：每个聊天窗口维护自己的 conversation_id，每次请求创建独立的 AbortController，每个流式响应只更新对应窗口的消息。

- Dify 负责： 对话上下文、会话标识、模型生成、流式响应。
- 前端负责： 多窗口状态隔离、请求生命周期管理、消息更新、任务取消、切换和恢复。

## 一、整体设计

### 纯前端多会话架构

![](data:image/png;base64,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)

## 二、Dify 会话机制

Dify 的 Chat API 核心接口是：

```
POST /v1/chat-messages
```

请求参数：

```
{
  "inputs": {},
  "query": "什么是 RAG？",
  "response_mode": "streaming",
  "user": "user_001",
  "conversation_id": ""
}
```

其中两个字段特别重要：

| 字段                | 作用                   |
| ----------------- | -------------------- |
| `user`            | 标识最终用户，多个窗口使用同一个用户标识 |
| `conversation_id` | 标识具体聊天会话，不同窗口使用不同 ID |

例如：

```
用户 user_001
  |
  ├── 窗口 A
  │    └── conversation_id = conv_A
  │
  ├── 窗口 B
  │    └── conversation_id = conv_B
  │
  └── 窗口 C
       └── conversation_id = conv_C
```

第一次发送消息时，不传 `conversation_id`，Dify 会创建新会话，并在流式事件中返回会话 ID。之后这个窗口的所有问题都使用这个 ID。

注意：不要每次提问都传空字符串，否则每次都可能创建新的会话，上下文就无法连续。

Dify 的接口也提供独立的 `task_id`，用于停止某一次正在执行的生成任务。



## 三、前端数据结构设计

会话和生成任务分开管理。

```
interface ChatSession {
  id: string;                  // 前端会话 ID
  conversationId: string|null; // Dify 会话 ID
  title: string;
  messages: ChatMessage[];
  status: 'idle' | 'streaming' | 'error';
}

interface ChatMessage {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  status: 'streaming' | 'completed' | 'error';
  taskId?: string;
}

interface RunningTask {
  taskId?: string; 
  sessionId: string;
  messageId: string;
  controller: AbortController;
}
```

同时维护两个 ID：

- `sessionId`：自己前端生成，用于管理聊天窗口。
- `conversationId`：Dify 返回，用于管理 Dify 上下文。


## 四、核心代码实现

前端技术栈采用 Vue 3 + Pinia + TypeScript

### 4.1 创建聊天 Store

```typescript
import { defineStore } from 'pinia';

export const useChatStore = defineStore('chat', {
  state: () => ({
    sessions: {} as Record<string, ChatSession>,
    activeSessionId: '',
    tasks: {} as Record<string, RunningTask>
  }),

  getters: {
    currentSession(state) {
      return state.sessions[state.activeSessionId];
    }
  },

  actions: {
    createSession() {
      const id = crypto.randomUUID();

      this.sessions[id] = {
        id,
        conversationId: null,
        title: '新对话',
        messages: [],
        status: 'idle'
      };

      this.activeSessionId = id;

      return id;
    },

    switchSession(id: string) {
      this.activeSessionId = id;
    },

    updateMessage(
      sessionId: string,
      messageId: string,
      content: string
    ) {
      const session = this.sessions[sessionId];
      if (!session) return;

      const message = session.messages.find(
        m => m.id === messageId
      );

      if (message) {
        message.content += content;
      }
    }
  }
});
```

### 4.2 调用 Dify 流式接口

将 Dify 请求封装成一个独立的 `DifyService`。

```typescript
const DIFY_BASE_URL = 'dify地址';
const DIFY_API_KEY = 'app-xxxxxxxx';

async function sendToDify(
  sessionId: string,
  query: string
) {
  const store = useChatStore();
  const session = store.sessions[sessionId];

  const userMessageId = crypto.randomUUID();
  const assistantMessageId = crypto.randomUUID();
  const controller = new AbortController();

  // 创建用户消息
  session.messages.push({
    id: userMessageId,
    role: 'user',
    content: query,
    status: 'completed'
  });

  // 创建 AI 占位消息
  session.messages.push({
    id: assistantMessageId,
    role: 'assistant',
    content: '',
    status: 'streaming'
  });

  const task: RunningTask = {
    sessionId,
    messageId: assistantMessageId,
    controller
  };

  store.tasks[assistantMessageId] = task;
  session.status = 'streaming';

  try {
    const response = await fetch(
      `${DIFY_BASE_URL}/chat-messages`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${DIFY_API_KEY}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          inputs: {},
          query,
          response_mode: 'streaming',
          user: 'user_001',
          conversation_id: session.conversationId || undefined
        }),
        signal: controller.signal
      }
    );

    if (!response.ok || !response.body) {
      throw new Error(`Dify 请求失败: ${response.status}`);
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    let buffer = '';

    while (true) {
      const { done, value } = await reader.read();

      if (done) break;

      buffer += decoder.decode(value, { stream: true });

      const events = buffer.split('\n\n');
      buffer = events.pop() || '';

      for (const rawEvent of events) {
        const dataLine = rawEvent
          .split('\n')
          .find(line => line.startsWith('data:'));

        if (!dataLine) continue;

        const data = JSON.parse(
          dataLine.slice(5).trim()
        );

        // 保存 Dify 会话 ID
        if (data.conversation_id && !session.conversationId) {
          session.conversationId = data.conversation_id;
        }

        // 保存 Dify 任务 ID
        if (data.task_id && !task.taskId) {
          task.taskId = data.task_id;
        }

        // 普通 Chatbot / Chatflow 文本输出
        if (data.event === 'message') {
          store.updateMessage(
            sessionId,
            assistantMessageId,
            data.answer || ''
          );
        }

        // Agent 应用文本输出
        if (data.event === 'agent_message') {
          store.updateMessage(
            sessionId,
            assistantMessageId,
            data.answer || ''
          );
        }

        // 结束事件
        if (
          data.event === 'message_end' ||
          data.event === 'workflow_finished'
        ) {
          const message = session.messages.find(
            m => m.id === assistantMessageId
          );

          if (message) message.status = 'completed';
        }

        // 错误事件
        if (data.event === 'error') {
          throw new Error(data.message || 'Dify 生成失败');
        }
      }
    }

  } catch (error: any) {
    const message = session.messages.find(
      m => m.id === assistantMessageId
    );

    if (error.name === 'AbortError') {
      if (message) message.status = 'completed';
    } else {
      if (message) message.status = 'error';
      console.error(error);
    }

  } finally {
    delete store.tasks[assistantMessageId];

    if (
      !session.messages.some(m => m.status === 'streaming')
    ) {
      session.status = 'idle';
    }
  }
}
```


### 4.3 停止某个窗口的生成任务

Dify 接口：

```
POST /v1/chat-messages/{task_id}/stop
```

前端实现：

```typescript
async function stopGeneration(messageId: string) {
  const store = useChatStore();

  const task = store.tasks[messageId];
  if (!task) return;

  // 1. 通知 Dify 停止模型生成
  if (task.taskId) {
    try {
      await fetch(
        `${DIFY_BASE_URL}/chat-messages/${task.taskId}/stop`,
        {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${DIFY_API_KEY}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            user: 'user_001'
          })
        }
      );
    } catch (error) {
      console.error('通知 Dify 停止失败', error);
    }
  }

  // 2. 关闭当前 SSE 连接
  task.controller.abort();

  // 3. 更新对应消息状态
  const session = store.sessions[task.sessionId];

  const message = session?.messages.find(
    m => m.id === messageId
  );

  if (message) {
    message.status = 'completed';
  }
}
```

注意：Dify 的停止接口需要使用对应应用类型的参数，`user` 应与创建任务时一致。前端 `AbortController.abort()` 和 Dify 停止接口是两个不同动作，不能只做前者。


### 4.4 切换窗口

```typescript
function switchSession(sessionId: string) {
  const store = useChatStore();

  // 只切换当前展示的会话
  store.activeSessionId = sessionId;

  // 不取消任何其他窗口的请求
}
```

例如：

```text
窗口 A：conversation_id = conv_A
       task_A 正在生成

窗口 B：conversation_id = conv_B
       task_B 正在生成

用户从 A 切换到 B：

只改变 activeSessionId

task_A 继续生成
task_B 继续生成
```

这里一定不要在组件销毁时调用 `abort()`。

## 五、保存多个窗口的数据

由于后端没有自己的数据库，使用浏览器的 IndexedDB，因为聊天内容可能非常多。

数据结构：

```typescript
interface LocalChatData {
  sessions: ChatSession[];
  activeSessionId: string;
  lastUpdatedAt: number;
}
```

数据关系：

```
IndexedDB
  |
  ├── Session A
  │    ├── conversationId
  │    ├── Messages
  │    └── Status
  │
  ├── Session B
  │    ├── conversationId
  │    ├── Messages
  │    └── Status
  │
  └── Session C
       ├── conversationId
       ├── Messages
       └── Status
```

注意：

- IndexedDB 保存前端展示的数据。
- Dify 保存实际的会话历史。
- 两者通过 `conversation_id` 关联。

刷新页面时：

1. 从 IndexedDB 恢复会话列表。
2. 恢复每个窗口的历史消息。
3. 恢复 `conversation_id`。
4. 用户切换到某个窗口时，可以通过 Dify 的会话消息查询 API 同步历史。
5. 不要直接假设之前的 SSE 连接还能恢复。

特别提醒：Dify 普通 Chatbot/Chatflow 的流式回答断线后，并没有通用的 Token 级断点续传接口。 可以查询已保存的会话消息，但如果需要无缝恢复尚未完成的生成过程，需要另外设计任务恢复机制。Workflow/Chatflow 的工作流事件重连能力则有所不同。



## 六、纯前端方案的一个重要安全问题

如果直接在浏览器中调用 Dify：

```
const DIFY_API_KEY = 'app-xxxxxxxx';
```

那么 API Key 会暴露在浏览器中。

用户可以通过：

- 浏览器开发者工具。
- Network 请求。
- 前端打包后的 JS 文件。

获取这个 Key。

可能造成其他人盗用你的 Dify 应用额度，甚至利用你的应用发起大量请求。

因此有两种部署模式：

| 方案             | 优点             | 缺点           |
| -------------- | -------------- | ------------ |
| 前端直接调用 Dify    | 不需要开发后端，实施简单   | API Key 暴露   |
| Dify WebApp 嵌入 | 无需自己管理 API Key | 页面定制和交互自由度受限 |
| 前端 + 轻量 API 代理 | 可以保护 Key，控制访问  | 需要一个简单代理服务   |
