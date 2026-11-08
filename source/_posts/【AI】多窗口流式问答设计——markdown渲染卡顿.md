---
title: 【AI】多窗口流式问答设计——markdown渲染卡顿
date: 2026-09-17 15:38:14
tags:
    - 多窗口问答
    - markdown渲染
---

# 【AI】多窗口流式问答设计——markdown渲染卡顿

要确保多个聊天窗口切换时 Markdown 渲染不会造成卡顿，关键不是简单地优化 Markdown 解析速度，而是要解决三个问题：

1. 渲染隔离： A 窗口收到 Token 时，不能导致 B 窗口重新渲染。
2. 增量渲染： 不要每收到一个 Token，就重新解析整篇 Markdown。
3. 渲染缓存： 切换窗口时，已经完成的 Markdown 内容不应该重新解析。

技术栈：Vue 3 + Pinia + Dify SSE

核心思想：会话状态隔离 + 消息组件隔离 + Markdown 增量渲染 + 请求级更新节流。

## 一、整体优化架构

### Markdown 渲染优化架构

![](data:image/png;base64,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)

## 二、聊天窗口组件隔离

不要把所有窗口的 Markdown 渲染都写在一个大型聊天组件中。

进行拆分：

```
ChatPage
 ├── ConversationList
 └── ChatContainer
      ├── ChatWindow A
      │    ├── MessageItem
      │    └── MarkdownRenderer
      ├── ChatWindow B
      │    ├── MessageItem
      │    └── MarkdownRenderer
      └── ChatWindow C
           ├── MessageItem
           └── MarkdownRenderer
```

Vue 中使用 `v-show` 控制窗口显示，而不是频繁销毁组件：

```vue
<template>
  <div
    v-for="session in sessions"
    :key="session.id"
    v-show="activeSessionId === session.id"
  >
    <ChatWindow
      :session-id="session.id"
    />
  </div>
</template>
```

这样切换窗口时，已经挂载的组件可以保留内部状态。

**注意**：`v-show` 只隐藏组件，并不会自动阻止它的响应式更新。因此还需要后面的消息级隔离。

如果会话数量很多，不要一次挂载全部窗口，使用 KeepAlive 或只保留当前窗口和少量最近访问窗口。

## 三、消息级组件隔离

使用独立的 MessageItem 组件：

```vue
<!-- ChatWindow.vue -->
<template>
  <div class="chat-window">
    <MessageItem
      v-for="message in messages"
      :key="message.id"
      :message="message"
    />
  </div>
</template>
```


```vue
// MessageItem.vue
<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  message: ChatMessage
}>();

const content = computed(() => props.message.content);
</script>

<template>
  <div class="message-item">
    <MarkdownRenderer
      :content="content"
      :streaming="message.status === 'streaming'"
    />
  </div>
</template>
```

**注意**：

- MessageItem 使用稳定的 messageId 作为 key。
- 不要给所有消息传递一个不断变化的全局对象。
- 不要在父组件中计算所有消息的 Markdown HTML。
- 不要因为一个消息变化而重建所有消息组件。

## 四、Markdown 增量渲染

解决长文本流式输出卡顿的核心。

假设 AI 正在生成一篇 10000 字的回答。

传统方式：

```typescript
watch(content, (value) => {
  html.value = marked.parse(value);
});
```

每收到一个 Token，就重新解析全部内容。

假设最终有 3000 次 Token 更新，那么可能产生 3000 次完整 Markdown 解析。

这会造成明显的性能浪费。

### 4.1 缓冲区 + 节流渲染

将高频 Token 更新合并为每 50ms 一次。

```vue
import { ref } from 'vue';
import { marked } from 'marked';

const content = ref('');
const renderedHtml = ref('');

let buffer = '';
let timer: number | null = null;

function appendToken(token: string) {
  buffer += token;

  if (timer !== null) return;

  timer = window.setTimeout(() => {
    content.value += buffer;
    buffer = '';

    renderedHtml.value = marked.parse(content.value) as string;

    timer = null;
  }, 50);
}
```

不过，这仍然是全量 Markdown 解析，只是减少了解析次数。

对于长文本，我更推荐下面的进一步优化。

### 4.2 已完成块缓存 + 当前块增量渲染

Markdown 内容天然可以按照块进行组织：

- 普通段落
- 标题
- 列表
- 引用
- 代码块
- 表格

把已经结束的 Markdown 块进行缓存，只重新解析最后一个尚未完成的块。

例如：

```
AI 正在输出：

# RAG 架构

RAG 主要由以下几个部分组成：

1. 文档解析
2. 向量化
3. 向量检索

目前正在生成：
4. 重排序
```

可以将前三部分作为已经完成的块缓存，只对最后正在生成的部分重新渲染。

伪代码：

```typescript
interface MarkdownBlock {
  id: string;
  source: string;
  html: string;
  completed: boolean;
}

const blocks: MarkdownBlock[] = [];

function updateMarkdownBlock(
  block: MarkdownBlock,
  newContent: string
) {
  block.source += newContent;

  if (isBlockComplete(block.source)) {
    block.html = marked.parse(block.source) as string;
    block.completed = true;
  } else {
    block.html = marked.parse(block.source) as string;
  }
}
```

这里的 `isBlockComplete()` 不能只判断是否出现换行符。生产实现需要识别 Markdown 块边界，并正确处理尚未闭合的代码围栏、表格、列表和引用。

如果要支持复杂 Markdown，我建议使用专门的流式 Markdown 解析器，或者自己实现可靠的块级状态机，而不是简单通过正则表达式切分。

## 五、切换窗口时不重新渲染 Markdown

增加一个 Markdown 缓存层。

```typescript
interface MarkdownCache {
  messageId: string;
  source: string;
  html: string;
  version: number;
  completed: boolean;
}
```

基本逻辑：

```typescript
const markdownCache = new Map<string, MarkdownCache>();

function renderMessage(message: ChatMessage) {
  const cache = markdownCache.get(message.id);

  if (
    cache &&
    cache.source === message.content &&
    cache.completed === (message.status === 'completed')
  ) {
    return cache.html;
  }

  const html = marked.parse(message.content) as string;

  markdownCache.set(message.id, {
    messageId: message.id,
    source: message.content,
    html,
    version: (cache?.version || 0) + 1,
    completed: message.status === 'completed'
  });

  return html;
}
```

缓存策略：

| 消息状态    | Markdown 策略    |
| ------- | -------------- |
| 正在生成    | 节流 + 当前块重新解析   |
| 生成完成    | 全量解析一次，缓存 HTML |
| 已完成且未修改 | 直接使用缓存         |
| 编辑或重新生成 | 清理对应消息缓存       |
| 切换窗口    | 不重新解析          |

**注意**：切换窗口时尽量不要重新创建 MarkdownRenderer。

## 六、处理代码块、表格等复杂内容

大模型回答中经常有：

```python
# 标题

普通段落

``
print("Hello World")
``

| 名称    | 说明  |
| ----- | --- |
| Redis | 缓存  |
| MySQL | 数据库 |

```

这些内容在流式生成时比较特殊。

例如，代码块尚未收到结束的三个反引号时，Markdown 解析器可能无法正确判断代码块是否结束。

处理：

| Markdown 类型 | 处理策略 |
|---|---|
| 普通文本 | 按段落增量渲染 |
| 标题 | 检测换行后确定块完成 |
| 列表 | 当前列表整体更新 |
| 代码块 | 闭合后再做最终高亮 |
| 表格 | 当前表格完成后再做最终渲染 |
| 数学公式 | 公式边界完整后再进行 KaTeX 渲染 |

注意：代码高亮，先显示普通代码文本，代码块闭合后再执行语法高亮。