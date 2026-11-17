---
title: 【RAG】分块策略
date: 2026-11-03 15:38:14
tags:
    - RAG
    - 分块策略
---

分块策略在RAG中至关重要，目的是提高效率、相关性和上下文保持。分块能减少计算开销，增加检索相关信息的可能性，同时保持信息完整性。但也存在风险，如上下文丢失、冗余和不一致性。选择策略需考虑文档类型（结构化 vs 非结构化）、查询复杂性、资源可用性和期望结果（速度 vs 准确性 vs 上下文）。


策略名称 | 描述 | 优点 | 缺点 | 实施建议
---|---|---|---|---
固定长度分块 | 将文本按预定义长度（如基于令牌或字符）分割，适合简单文档或快速处理。 | 简单性、统一性、效率 | 上下文丢失、相关性问题、潜在信息丢失 | 选择合适大小，考虑使用重叠窗口以保持上下文
基于句子的分块 | 按句子边界分割，确保每个块是完整的思想，适合短响应如客户查询。 | 保持上下文、易于实现、更好的可读性 | 块大小不一致、不适合长句、控制力弱 | 使用NLP库检测句子，必要时合并短句
段落分块 | 将文档按段落分割，每个段落通常包含完整想法或主题，适合结构化文档。 | 更丰富的上下文、逻辑分割 | 大小不一致、可能超出行牌限制 | 监控块大小，必要时分割大段落以保持上下文
文档分块 | 将整个文档视为一个块或最小分割，保持完整结构和上下文，适合法律或医学文档。 | 保持完整上下文、适合结构化文本、简单 | 可扩展性问题、效率低、缺乏细节提取 | 适用于不希望分割影响流程的场景
滑动窗口分块 | 通过滑动窗口创建重叠块，确保相邻块共享内容，适合需保持跨部分上下文的文档。 | 上下文连续性、提高检索效果 | 冗余、计算成本高 | 优化窗口大小和重叠，使用去重技术处理冗余
语义分块 | 使用嵌入或机器学习模型按语义意义分割，确保每个块主题连贯。 | 上下文相关性、灵活性、提高检索准确性 | 复杂性、处理时间长、计算成本高、阈值调整 | 利用预训练模型，平衡成本和粒度
递归分块 | 使用层次分隔符或多个分隔符逐步分割为小块，适合大型结构化文档。 | 层次上下文、可扩展性、意义块、细粒度控制 | 实现复杂、可能丢失上下文、复杂性高、计算开销大 | 使用文档结构标识层次，存储块位置元数据
上下文增强分块 | 为每个块添加周围块的摘要或元数据，保持序列间的上下文，适合长文档。 | 增强上下文、提高连贯性 | 复杂性、存储开销大 | 生成简洁摘要，考虑用关键术语或概念作为元数据
模式特定分块 | 按内容类型（如文本、表格、图像）分别处理，每个类型按其性质分块。 | 定制方法、提高准确性 | 实现复杂、集成困难 | 使用OCR处理图像，将表格转为结构化数据，保持索引一致
代理分块 | 使用AI（如大型语言模型）按内容结构和语义建议块边界，或按代理角色任务组织。 | 任务导向效率、聚焦相关数据、灵活性、智能分割 | 复杂设置、计算密集、成本高、可能过分专业化、丢失全局上下文 | 选择性使用，优化LLM提示，明确代理角色和任务规则
子文档分块 | 总结整个文档或大节并将摘要作为元数据附加到单个块，适合大型文档集。 | 层次检索、上下文深度 | 额外处理、元数据管理复杂 | 自动化摘要生成，高效存储摘要
混合分块 | 结合多种分块策略，动态适应不同查询类型或文档结构，适合多用途系统。 | 灵活性、优化性能 | 逻辑复杂、维护难度大 | 制定选择策略的标准，广泛测试以确保可靠性
基于令牌的分块 | 按预定义令牌数（如单词或子词）分割，确保块在模型令牌限制内。 | 优化语言模型、精确控制块大小、一致性 | 上下文碎片化、忽略语义结构、灵活性有限 | 确保块在模型令牌限制内



## 1. 固定长度分块（Fixed-size Chunking）
实现原理： 按照固定字符数切分文本，可以设置重叠长度，避免切分位置丢失上下文。

使用 LangChain 的 CharacterTextSplitter。

fixed.py
```python
from langchain_text_splitters import CharacterTextSplitter
from langchain_core.documents import Document


class FixedChunker:

    def __init__(
        self,
        chunk_size=500,
        chunk_overlap=100
    ):
        self.splitter = CharacterTextSplitter(
            separator="",
            chunk_size=chunk_size,
            chunk_overlap=chunk_overlap,
            length_function=len,
            strip_whitespace=True
        )

    def split(self, document: Document):
        chunks = self.splitter.split_documents([document])

        for i, chunk in enumerate(chunks):
            chunk.metadata.update({
                "strategy": "fixed",
                "chunk_index": i
            })

        return chunks
```

app.py
```python
from langchain_core.documents import Document

doc = Document(
    page_content="""需要处理的文本""",
    metadata={"document_id": "doc001"}
)

chunker = FixedChunker(
    chunk_size=500,
    chunk_overlap=100
)

chunks = chunker.split(doc)

for chunk in chunks:
    print(chunk)
```


参数建议：
参数 | 推荐值 | 说明
---|---|---
chunk_size | 500 | 每块字符数
chunk_overlap | 100 | 重叠字符数
overlap比例 | 20% | 减少上下文断裂

注意：这里是字符长度，不是 Token 数量。

## 2. 基于句子的分块（Sentence Chunking）
实现原理： 先识别句子边界，再按照目标长度合并句子，尽量确保每个 Chunk 都包含完整的语义表达。

对于中文，需要同时识别 。！？ 等标点。

sentence.py

```python
import re
from langchain_core.documents import Document


class SentenceChunker:

    def __init__(self, chunk_size=500):
        self.chunk_size = chunk_size

    def split_sentences(self, text):

        pattern = r'(?<=[。！？；.!?;])\s*'

        return [
            s.strip()
            for s in re.split(pattern, text)
            if s.strip()
        ]

    def split(self, document: Document):

        sentences = self.split_sentences(
            document.page_content
        )

        chunks = []
        current = ""

        for sentence in sentences:

            if len(current) + len(sentence) <= self.chunk_size:
                current += sentence
            else:

                if current:
                    chunks.append(current)

                # 处理超长句子
                if len(sentence) > self.chunk_size:
                    for i in range(
                        0, len(sentence), self.chunk_size
                    ):
                        chunks.append(
                            sentence[i:i+self.chunk_size]
                        )
                    current = ""
                else:
                    current = sentence

        if current:
            chunks.append(current)

        return [
            Document(
                page_content=text,
                metadata={
                    **document.metadata,
                    "strategy": "sentence",
                    "chunk_index": i
                }
            )
            for i, text in enumerate(chunks)
        ]
```

app.py
```python
from langchain_core.documents import Document
from sentence import SentenceChunker

sentenceChunker = SentenceChunker()

document = Document(
    page_content="""需要处理的文本""",
    metadata={"document_id": "doc001"}
)
chunks = sentenceChunker.split(document=document)

for chunk in chunks:
    print(chunk)
```

这里有一个关键点：不要简单地一条句子对应一个 Chunk。

例如：
```
第一句：RAG是一种检索增强生成技术。
第二句：它通过外部知识库为大模型提供上下文。
第三句：能够降低模型产生幻觉的概率。
```


如果每句话单独分块，第二句话和第三句话可能无法独立表达完整含义。

因此，推荐采用句子识别 + 目标长度合并的方式。

## 3. 段落分块（Paragraph Chunking）
实现原理： 以换行符、空行或文档段落结构为依据，将完整段落作为基本分块单元。

如果段落过长，再使用递归分块进行二次拆分。

paragraph.py

```python
import re
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter
)
from langchain_core.documents import Document


class ParagraphChunker:

    def __init__(self, chunk_size=800, overlap=100):

        self.max_splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=overlap,
            separators=["。", "；", "，", " "]
        )

    def split(self, document: Document):

        paragraphs = re.split(
            r"\n\s*\n",
            document.page_content
        )

        chunks = []

        for paragraph in paragraphs:

            paragraph = paragraph.strip()

            if not paragraph:
                continue

            if len(paragraph) <= self.max_splitter._chunk_size:
                chunks.append(paragraph)
            else:
                chunks.extend(
                    self.max_splitter.split_text(paragraph)
                )

        return [
            Document(
                page_content=text,
                metadata={
                    **document.metadata,
                    "strategy": "paragraph",
                    "chunk_index": i
                }
            )
            for i, text in enumerate(chunks)
        ]
```


app.py
```python
from langchain_core.documents import Document
from paragraph import ParagraphChunker

Chunker = ParagraphChunker()

document = Document(
    page_content="""需要处理的文本""",
    metadata={"document_id": "doc001"}
)
chunks = Chunker.split(document=document)

for chunk in chunks:
    print(chunk)
```

实际生产环境中，建议从 PDF、Word 解析阶段就保留段落边界，不要完全依赖换行符，因为 PDF 提取出来的换行可能只是版面换行。

## 4. 文档分块（Document Chunking）
实现原理： 将整个文档视为一个整体，不进行拆分，保留完整的文档上下文。

这种方式适用于篇幅较短、需要完整上下文的文档。

strategies/document.py

```python
from langchain_core.documents import Document


class DocumentChunker:

    def split(self, document: Document):
        return [
            Document(
                page_content=document.page_content,
                metadata={
                    **document.metadata,
                    "strategy": "document",
                    "chunk_index": 0,
                    "is_full_document": True
                }
            )
        ]
```


注意：

文档级分块不意味着可以将几十万字的文件直接交给 Embedding 模型。

实际工程中，可以采用：
- 短文档：整篇文档向量化。
- 长文档：整篇文档保留为父文档，子章节进行向量化。
- 检索时：先检索子块，再通过 parent_id 获取完整文档。

## 5. 滑动窗口分块（Sliding Window Chunking）
实现原理： 使用固定窗口大小，按照固定步长向前滑动，生成具有重叠内容的 Chunk。

例如：
```
原文：
A B C D E F G H I J

窗口大小：5
滑动步长：3

Chunk1：A B C D E
Chunk2：D E F G H
Chunk3：G H I J
```

strategies/sliding_window.py
```python
from langchain_core.documents import Document


class SlidingWindowChunker:

    def __init__(
        self,
        window_size=500,
        step_size=400
    ):

        if window_size <= 0 or step_size <= 0:
            raise ValueError("窗口和步长必须大于0")

        if step_size > window_size:
            raise ValueError("步长不能大于窗口，避免内容遗漏")

        self.window_size = window_size
        self.step_size = step_size

    def split(self, document: Document):

        text = document.page_content
        chunks = []

        for start in range(
            0, len(text), self.step_size
        ):

            end = start + self.window_size

            chunk = text[start:end]

            if chunk.strip():
                chunks.append({
                    "text": chunk,
                    "start": start,
                    "end": min(end, len(text))
                })

            if end >= len(text):
                break

        return [
            Document(
                page_content=item["text"],
                metadata={
                    **document.metadata,
                    "strategy": "sliding_window",
                    "chunk_index": i,
                    "start_offset": item["start"],
                    "end_offset": item["end"]
                }
            )
            for i, item in enumerate(chunks)
        ]
```


这种方式适合：
- 长篇访谈记录。
- 会议记录。
- 法律法规。
- 医疗病例。
- 需要保持前后文关系的长文本。
这里的窗口和步长均按字符计算。

## 6. 语义分块（Semantic Chunking）

实现原理： 不再按照固定字符长度切分，而是根据相邻句子的语义相似度判断是否需要切分。

基本流程：
1. 将文本拆分成句子。
2. 使用 Embedding 模型将句子转为向量。
3. 计算相邻句子的余弦相似度。
4. 当相似度低于阈值时，判断为主题变化。
5. 在该位置切分。

semantic.py
```python
import numpy as np

from langchain_core.documents import Document
from langchain_ollama import OllamaEmbeddings


class SemanticChunker:

    def __init__(
        self,
        embeddings=None,
        threshold=0.70,
        min_chunk_size=100,
        max_chunk_size=1000
    ):

        self.embeddings = embeddings or OllamaEmbeddings(
            model="quentinz/bge-large-zh-v1.5:latest",
            base_url="http://127.0.0.1:11434"
        )

        self.threshold = threshold
        self.min_chunk_size = min_chunk_size
        self.max_chunk_size = max_chunk_size

    @staticmethod
    def cosine_similarity(a, b):

        a = np.array(a)
        b = np.array(b)

        denominator = (
            np.linalg.norm(a) * np.linalg.norm(b)
        )

        if denominator == 0:
            return 0.0

        return float(np.dot(a, b) / denominator)

    def split(self, document: Document):

        sentences = [
            s.strip()
            for s in document.page_content.replace(
                "\n", " "
            ).split("。")
            if s.strip()
        ]

        sentences = [s + "。" for s in sentences]

        if not sentences:
            return []

        vectors = self.embeddings.embed_documents(
            sentences
        )

        chunks = []
        current = sentences[0]

        for i in range(1, len(sentences)):

            similarity = self.cosine_similarity(
                vectors[i-1],
                vectors[i]
            )

            candidate = current + sentences[i]

            should_split = (
                similarity < self.threshold
                and len(current) >= self.min_chunk_size
            )

            should_split |= (
                len(candidate) > self.max_chunk_size
            )

            if should_split:

                chunks.append(current)
                current = sentences[i]

            else:
                current = candidate

        if current:
            chunks.append(current)

        return [
            Document(
                page_content=text,
                metadata={
                    **document.metadata,
                    "strategy": "semantic",
                    "chunk_index": i
                }
            )
            for i, text in enumerate(chunks)
        ]
```

这里展示的是基础语义分块实现。生产环境还需要考虑：
- 中文标点和英文标点混合。
- 相邻句子向量的批量计算。
- 使用相似度百分位数动态确定阈值。
- 最小块和最大块长度。
- 对超长句子进行递归拆分。
- 语义边界与标题、段落边界结合。

LangChain 实现：
```python
from langchain_experimental.text_splitter import (
    SemanticChunker as LangChainSemanticChunker
)

from langchain_openai import OpenAIEmbeddings

splitter = LangChainSemanticChunker(
    OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile",
    breakpoint_threshold_amount=80
)

chunks = splitter.split_documents([doc])
```

**注意**：安装对应版本的 langchain-experimental，`pip install -U langchain-experimental`


## 7. 递归分块（Recursive Chunking）

**实现原理**： 按照不同层级的分隔符依次尝试切分。

例如：
```
第一优先级：章节标题
第二优先级：段落
第三优先级：句子
第四优先级：空格
第五优先级：字符
```

当使用高优先级分隔符无法得到合适大小的 Chunk 时，继续使用低优先级分隔符。

recursive.py
```python
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter
)


class RecursiveChunker:

    def __init__(
        self,
        chunk_size=800,
        chunk_overlap=100
    ):

        self.splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=chunk_overlap,
            separators=[
                "\n# ",
                "\n## ",
                "\n### ",
                "\n\n",
                "\n",
                "。",
                "！",
                "？",
                "；",
                "，",
                " ",
                ""
            ],
            length_function=len,
            keep_separator=True
        )

    def split(self, document):

        chunks = self.splitter.split_documents(
            [document]
        )

        for i, chunk in enumerate(chunks):

            chunk.metadata.update({
                "strategy": "recursive",
                "chunk_index": i
            })

        return chunks
```


对于中文知识库，可以根据文档类型调整分隔符。

例如：
```python
separators = [
    "\n第",
    "\n一、",
    "\n（一）",
    "\n1.",
    "\n\n",
    "\n",
    "。",
    "；",
    "，",
    ""
]
```



**注意**：递归分隔符只能提供切分优先级，并不天然保证标题和正文之间的父子关系。正式入库时，建议额外解析章节树，并将章节路径写入 metadata。

## 8. 上下文增强分块（Contextual Chunking）
**实现原理**： 在原始 Chunk 的基础上，额外增加上下文信息，例如：
- 所属文档名称。
- 所属章节。
- 前一个 Chunk 的摘要。
- 后一个 Chunk 的摘要。
- 当前 Chunk 的主题。
- 关键实体和术语。


例如原文：
```
文档名称：企业知识管理规范
章节：第三章 / 知识审核

Chunk：
审核人员需要对提交的知识进行审核。
```

增强后：
```
文档：企业知识管理规范
章节：第三章 / 知识审核
前文摘要：本章介绍知识入库和审核流程。
当前内容：审核人员需要对提交的知识进行审核。
后文摘要：审核通过后，知识进入正式知识库。
```

contextual.py
```python
from langchain_core.documents import Document
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI


class ContextualChunker:

    def __init__(self, llm=None):

        self.llm = llm or ChatOpenAI(
            model="gpt-4o-mini",
            temperature=0
        )

        self.prompt = ChatPromptTemplate.from_template("""
你是一个知识库上下文增强助手。

请根据文档信息，为当前文本块补充必要的上下文。

文档名称：{document_name}
章节：{section}

前一个文本块：
{previous}

当前文本块：
{current}

后一个文本块：
{next}

要求：
1. 不得改变当前文本块的事实。
2. 补充文档背景、所属章节和必要的指代信息。
3. 不要重复大段原文。
4. 不得引入原文没有的事实。
5. 直接输出增强后的文本。
""")

    def split(self, document):

        # 先进行基础分块
        from langchain_text_splitters import (
            RecursiveCharacterTextSplitter
        )

        splitter = RecursiveCharacterTextSplitter(
            chunk_size=500,
            chunk_overlap=50
        )

        chunks = splitter.split_documents([document])

        results = []

        for i, chunk in enumerate(chunks):

            previous = (
                chunks[i-1].page_content
                if i > 0 else "无"
            )

            next_text = (
                chunks[i+1].page_content
                if i < len(chunks)-1 else "无"
            )

            chain = self.prompt | self.llm

            response = chain.invoke({
                "document_name": document.metadata.get(
                    "source", "未知文档"
                ),
                "section": document.metadata.get(
                    "section", "未知章节"
                ),
                "previous": previous[:300],
                "current": chunk.page_content,
                "next": next_text[:300]
            })

            results.append(
                Document(
                    page_content=chunk.page_content,
                    metadata={
                        **chunk.metadata,
                        "strategy": "contextual",
                        "contextual_content": response.content,
                        "chunk_index": i
                    }
                )
            )

        return results
```



在实际使用中时，原始内容与增强内容分开存储：
- page_content：原始文本。
- contextual_content：增强后的检索文本。

向量化时可以选择使用增强内容，但最终答案引用时应返回原始文本，避免模型生成的补充描述被误认为原文证据。

**缺点**：需要使用Embedding模型，成本过高，

## 9. 模式特定分块（Schema-specific Chunking）
**实现原理**： 根据不同内容类型使用不同的分块方法。

例如：
内容类型 | 处理策略
---|---
普通文本 | 递归分块
表格 | 按行、逻辑记录分块
图片 | OCR + 图片描述
代码 | 按类、函数切分
JSON | 按对象结构切分
Markdown | 按标题层级切分

特别是表格，不能直接将其转换成普通文本后随意切分。

schema.py
```python
import json
import pandas as pd

from langchain_core.documents import Document
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter
)


class SchemaChunker:

    def __init__(self, chunk_size=800):

        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=100
        )

    def split_text(self, document):

        return self.text_splitter.split_documents([document])

    def split_table(
        self,
        dataframe: pd.DataFrame,
        metadata
    ):

        columns = list(dataframe.columns)

        chunks = []
        current_rows = []

        for _, row in dataframe.iterrows():

            current_rows.append(
                row.to_dict()
            )

            # 简单按行数控制
            if len(current_rows) >= 10:

                chunks.append(
                    self._table_document(
                        columns,
                        current_rows,
                        metadata
                    )
                )

                current_rows = []

        if current_rows:

            chunks.append(
                self._table_document(
                    columns,
                    current_rows,
                    metadata
                )
            )

        return chunks

    @staticmethod
    def _table_document(
        columns,
        rows,
        metadata
    ):

        content = {
            "columns": columns,
            "rows": rows
        }

        return Document(
            page_content=json.dumps(
                content,
                ensure_ascii=False,
                default=str
            ),
            metadata={
                **metadata,
                "strategy": "table",
                "content_type": "table",
                "columns": columns
            }
        )
```


表格处理示例：
```python
import pandas as pd

df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五"],
    "年龄": [25, 30, 18],
    "部门": ["技术部", "销售部", "研发部"]
})

chunker = SchemaChunker()

chunks = chunker.split_table(
    df,
    metadata={
        "document_id": "table001",
        "table_name": "员工信息表"
    }
)
```



这里采用的是基础按行切分。对于复杂 Excel、Word 表格，还需要额外实现：
- 合并单元格还原。
- 多级表头识别。
- 跨页表格拼接。
- 表格标题和说明关联。
- 行列之间的层级关系保留。
例如，一个表格中包含合并单元格时，需要先将合并区域的语义信息传播到对应行，再生成 Chunk，而不是直接按照物理行数切分。

## 10. 代理分块（Agent-based Chunking）
**实现原理**： 通过 LLM 分析文档内容，识别章节、主题、语义边界，并输出建议的分块结果。

与普通语义分块的区别：
- 语义分块：主要依据 Embedding 相似度判断边界。
- 代理分块：通过 LLM 理解文本结构、任务目标和领域规则，再决定边界。
推荐使用 LangChain 的 with_structured_output()，避免 LLM 返回不可解析的自由文本。

agentBased.py
```python
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.documents import Document


class AgentChunk(BaseModel):

    title: str = Field(
        description="文本块标题"
    )

    content: str = Field(
        description="完整文本块内容"
    )

    topic: str = Field(
        description="文本块主题"
    )

    keywords: list[str] = Field(
        description="关键词"
    )


class AgentChunkResult(BaseModel):

    chunks: list[AgentChunk]


class AgentChunker:

    def __init__(self, llm=None):

        self.llm = llm or ChatOpenAI(
            model="gpt-4o",
            temperature=0
        )

        self.structured_llm = (
            self.llm.with_structured_output(
                AgentChunkResult
            )
        )

        self.prompt = ChatPromptTemplate.from_messages([
            (
                "system",
                """
你是一个专业的文档分块Agent。

请按照语义完整性和文档结构拆分输入文本。

规则：
1. 保留原始文本，不得编造内容。
2. 不要拆开完整的逻辑论述。
3. 标题必须与正文对应。
4. 每个文本块建议控制在200~800字。
5. 不得遗漏原文内容。
6. 输出结构化的分块结果。
"""
            ),
            (
                "human",
                "请分块以下内容：\n{text}"
            )
        ])

    def split(self, document: Document):

        chain = self.prompt | self.structured_llm

        result = chain.invoke({
            "text": document.page_content
        })

        return [
            Document(
                page_content=item.content,
                metadata={
                    **document.metadata,
                    "strategy": "agent",
                    "title": item.title,
                    "topic": item.topic,
                    "keywords": item.keywords,
                    "chunk_index": i
                }
            )
            for i, item in enumerate(result.chunks)
        ]
```


**注意**：LLM 的输出不一定能够保证原文完整无遗漏。

生产环境需要增加校验机制：
1. 每个输出块必须能在原文中找到对应内容。
2. 检查块之间是否存在遗漏。
3. 检查是否出现重复文本。
4. 对无法通过校验的文档回退到递归分块。
5. 限制单次输入长度，超长文档先按章节拆分。

对于法律、医疗等高准确性知识库，不建议直接使用 LLM 重写原文作为最终证据。

## 11. 子文档分块（Parent-Child Chunking）
**实现原理**： 将文档分为父级 Chunk 和子级 Chunk。
- 父级 Chunk：保留较大的章节或文档上下文。
- 子级 Chunk：按照较小粒度切分，用于向量检索。
- 检索命中子块后，通过父级 ID 找到对应父块。


例如：
```
企业知识管理规范
│
├── 父块1：第一章 知识管理概述
│   ├── 子块1.1：知识管理定义
│   ├── 子块1.2：知识管理目标
│   └── 子块1.3：知识管理原则
│
└── 父块2：第二章 知识审核
    ├── 子块2.1：审核流程
    ├── 子块2.2：审核人员职责
    └── 子块2.3：审核标准
```

subdocument.py
```python
import uuid

from langchain_core.documents import Document
from langchain_text_splitters import (
    RecursiveCharacterTextSplitter
)


class ParentChildChunker:

    def __init__(
        self,
        parent_size=2000,
        child_size=500
    ):

        self.parent_splitter = RecursiveCharacterTextSplitter(
            chunk_size=parent_size,
            chunk_overlap=100
        )

        self.child_splitter = RecursiveCharacterTextSplitter(
            chunk_size=child_size,
            chunk_overlap=50
        )

    def split(self, document: Document):

        parents = self.parent_splitter.split_documents(
            [document]
        )

        parent_docs = []
        child_docs = []

        for index, parent in enumerate(parents):

            parent_id = str(uuid.uuid4())

            parent.metadata.update({
                "parent_id": parent_id,
                "chunk_type": "parent",
                "parent_index": index
            })

            parent_docs.append(parent)

            children = self.child_splitter.split_documents(
                [parent]
            )

            for child_index, child in enumerate(children):

                child.metadata.update({
                    "parent_id": parent_id,
                    "chunk_type": "child",
                    "child_index": child_index
                })

                child_docs.append(child)

        return {
            "parents": parent_docs,
            "children": child_docs
        }
```

LangChain 官方：
```python
from langchain_chroma import Chroma
from langchain_core.documents import Document
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore

# 1. 准备原始文档
docs = [
    Document(page_content="这里是一篇很长的示例文档，包含了很多细节内容……（省略长文本）")
]

# 2. 定义父块切分器（大块，例如：1500字符）
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=1500, chunk_overlap=150)

# 3. 定义子块切分器（小块，例如：300字符）
child_splitter = RecursiveCharacterTextSplitter(chunk_size=300, chunk_overlap=30)

# 4. 初始化向量数据库和底层Docstore
vectorstore = Chroma(
    collection_name="split_parents", embedding_function=OpenAIEmbeddings()
)
store = InMemoryStore()  # 用于存放父文档内容

# 5. 初始化 ParentDocumentRetriever
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)

# 6. 添加文档
retriever.add_documents(docs)

# 7. 检索测试（会自动返回对应的父文档大块）
retrieved_docs = retriever.invoke("你的查询问题是什么？")
```


检索时只需要对 children 建立向量索引。

命中后通过 parent_id 获取 parents 中对应的父级内容。

这种方法非常适合长篇法律文档、医疗指南、企业制度和技术手册。

## 12. 混合分块（Hybrid Chunking）

生产环境中使用

**实现原理**： 不同类型的文档采用不同分块策略，而不是所有文档使用同一种切分方式。

例如：
文档类型 | 推荐策略
---|---
FAQ | 句子 + 段落
技术文档 | 递归 + 子文档
法律法规 | 章节 + 子文档
医疗病例 | 模式特定 + 语义
Excel | 表格分块
会议记录 | 时间窗口 + 语义
普通 PDF | 递归分块
超长文档 | 父子分块

采用的是规则路由方式进行分类，依靠`Document`中`metadata.doc_type`判断当前文档的类型，根据类型选择不同的分块策略

hybrid.py
```python
from langchain_core.documents import Document


class HybridChunker:

    def __init__(self):

        from sentence import SentenceChunker
        from recursive import RecursiveChunker
        from subdocument import ParentChildChunker
        from schema import SchemaChunker

        self.sentence = SentenceChunker(
            chunk_size=500
        )

        self.recursive = RecursiveChunker(
            chunk_size=800,
            chunk_overlap=100
        )

        self.parent_child = ParentChildChunker()

        self.schema = SchemaChunker()

    def split(self, document: Document):

        doc_type = document.metadata.get(
            "document_type",
            "text"
        )

        if doc_type == "faq":

            return self.sentence.split(document)

        elif doc_type in ["technical", "pdf"]:

            return self.recursive.split(document)

        elif doc_type in ["law", "medical"]:

            return self.parent_child.split(document)

        elif doc_type == "table":

            raise ValueError(
                "表格需要传入结构化数据，调用 SchemaChunker.split_table"
            )

        else:

            return self.recursive.split(document)
```


## 13. 基于 Token 的分块（Token-based Chunking）
**实现原理**： 使用 Token 数量作为分块长度，而不是字符数量。

**注意**：不同模型的 Tokenizer 不同，同样长度的文本可能产生不同的 Token 数量。

例如：
```
中文文本：不同的字符组合可能对应不同Token数量。
英文文本：一个单词可能拆分为多个Token。
```

token.py
```python
import tiktoken

from langchain_text_splitters import (
    RecursiveCharacterTextSplitter
)


class TokenChunker:

    def __init__(
        self,
        chunk_size=500,
        chunk_overlap=50,
        encoding_name="cl100k_base"
    ):

        self.encoding = tiktoken.get_encoding(
            encoding_name
        )

        self.splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=chunk_overlap,
            length_function=self.count_tokens,
            separators=[
                "\n\n",
                "\n",
                "。",
                "！",
                "？",
                "；",
                "，",
                " ",
                ""
            ]
        )

    def count_tokens(self, text):

        return len(
            self.encoding.encode(text)
        )

    def split(self, document):

        chunks = self.splitter.split_documents(
            [document]
        )

        for i, chunk in enumerate(chunks):

            chunk.metadata.update({
                "strategy": "token",
                "chunk_index": i,
                "token_count": self.count_tokens(
                    chunk.page_content
                )
            })

        return chunks
```


调用：
```python
from langchain_core.documents import Document
from TokenBased.tokenBased import TokenChunker

chunker = TokenChunker(
    chunk_size=500,
    chunk_overlap=50,
    encoding_name="cl100k_base"
)
document = Document(
    page_content="""这是需要处理的文本""",
    metadata={"source": "https://example.com"}
)
chunks = chunker.split(docuemnt)
```


**注意**：cl100k_base 只是一个 Tokenizer 编码示例。使用其他模型时，应当尽量选择与目标模型匹配的 Tokenizer。