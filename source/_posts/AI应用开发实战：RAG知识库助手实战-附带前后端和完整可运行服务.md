---
title: AI应用开发实战：RAG知识库助手实战-附带前后端和完整可运行服务
date: 2026-08-04 14:30:20
tags:

---

AI应用开发实战：RAG知识库助手实战-附带前后端和完整可运行服务 

![img](https://dqej47nflyz.feishu.cn/space/api/box/stream/download/asynccode/?code=Mzg0ZjliM2FhMzliZjQ1M2Y5MDIyMzUzMmVkNTI2ZWRfVGp5SkdNZmRTZ3c1MHZ0WE1NNWpBbDBvNXUxcVI2TjBfVG9rZW46SEtyTmIwYmNEb3NMTDV4ekdrUmNKUUQ0bjZiXzE3ODU4MjUwMzM6MTc4NTgyODYzM19WNA&add_watermark=true&scene_type=CCM)

# RAG 知识库问答系统 - 从零开始的完整指南

> 基于 Milvus 向量数据库 + LangChain + DashScope 的本地知识库问答解决方案

直接可以运行的代码已经整理好了，需要的话找老师拿

![img](https://dqej47nflyz.feishu.cn/space/api/box/stream/download/asynccode/?code=MjhiZDE0Yjg3ZDc4YzYyYWFjMDcxOWU4YTJmOTEyY2FfMFd0R0RHWElnT1FIWmNqVEt0OWpmb0NhZ3NRS1JxOWxfVG9rZW46U1RvN2JmWUdTb1VlbGN4Q2lTZ2NhQ2hFbjhlXzE3ODU4MjUwMzM6MTc4NTgyODYzM19WNA&add_watermark=true&scene_type=CCM)

1. ## 什么是 RAG？

### 1.1 RAG 的定义

**RAG（Retrieval-Augmented Generation，检索增强生成）** 是一种将**信息检索**与**大语言模型（LLM）生成**相结合的技术。

用一个简单的比喻来理解：

> 想象你在参加一场开卷考试。RAG 就像是：
>
> - 📚 **你的参考书**（知识库）
> - 🔍 **快速翻书找答案的能力**（检索系统）
> - 🧠 **理解并组织答案的大脑**（大语言模型）

### 1.2 为什么需要 RAG？

![img](https://dqej47nflyz.feishu.cn/space/api/box/stream/download/asynccode/?code=Yzg2NDdjMjM4YmYxOTA3OGJmMjdlNjk0YTQ4MmRmOTNfc2VZV3M3TGozOFR5RVY4WThRbVB2VThCb1J2TXZBT2NfVG9rZW46UWRsemJHWHNJbzRHeXV4cGpUVGNUZDE0bkNiXzE3ODU4MjUwMzM6MTc4NTgyODYzM19WNA&add_watermark=true&scene_type=CCM)

传统大语言模型存在几个问题：

| 问题     | 说明               | RAG 如何解决         |
| -------- | ------------------ | -------------------- |
| 知识过时 | 模型训练后不再更新 | 可以随时更新知识库   |
| 幻觉问题 | 模型可能"编造"答案 | 基于真实文档生成回答 |
| 领域限制 | 缺乏专业/私有知识  | 可导入任意专业文档   |
| 不可追溯 | 无法知道答案来源   | 可以引用原始文档     |

### 1.3 RAG 的工作流程

```Plain
┌─────────────────────────────────────────────────────────────┐
│                    RAG 完整工作流程                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  【索引阶段 - 离线处理】                                      │
│                                                             │
│  文档 → 加载 → 切分 → 向量化 → 存入向量数据库                  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  【检索阶段 - 在线查询】                                      │
│                                                             │
│  问题 → 向量化 → 相似度搜索 → 获取相关文档 → LLM生成答案       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

1. ## 什么是 Milvus？

### 2.1 向量数据库简介

在理解 Milvus 之前，我们需要先了解什么是**向量**：

**向量（Vector）** 是一组数字，用来表示文本、图片等数据的"语义特征"。

例如：

```Plain
"猫"  → [0.23, -0.45, 0.78, ..., 0.12]  (1536维向量)
"狗"  → [0.21, -0.42, 0.75, ..., 0.15]  (相似度高！)
"汽车" → [0.89, 0.23, -0.56, ..., 0.67]  (相似度低)
```

**向量数据库** 就是专门用来存储和快速搜索这些向量的数据库。

### 2.2 Milvus 是什么？

**Milvus** 是一个开源的向量数据库，专门为 AI 应用设计：

- 🚀 **高性能**：支持亿级向量的毫秒级搜索
- 📊 **可扩展**：支持分布式部署
- 🔧 **易集成**：提供 Python、Java、Go 等多种 SDK
- 💾 **持久化**：数据可靠存储，支持备份恢复

### 2.3 Milvus 的核心概念

| 概念       | 类比     | 说明                 |
| ---------- | -------- | -------------------- |
| Collection | 数据库表 | 存储向量数据的容器   |
| Entity     | 表中的行 | 一条向量数据记录     |
| Field      | 表中的列 | 向量字段、标量字段等 |
| Index      | 索引     | 加速搜索的数据结构   |

### 2.4 本项目中的 Milvus 配置

```Python
# 连接配置
MILVUS_HOST = "localhost"
MILVUS_PORT = "19530"

# 集合 Schema
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="name", dtype=DataType.VARCHAR, max_length=255),
    FieldSchema(name="text", dtype=DataType.VARCHAR, max_length=5000),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=1536)
]
```

1. ## 项目架构概览

### 3.1 整体架构图

本项目的架构可以分为两大流程：

**📥 文档索引流程（Indexing Pipeline）**

```Plain
本地文档 → DocumentLoader → TextSplitter → Embedding Model → Milvus
   ①            ②              ③              ④             ⑤
```

**🔍 检索问答流程（Retrieval Pipeline）**

```Plain
用户提问 → Query Embedding → Vector Search → Top-K Results → LLM → 答案
   ⑥            ⑦              ⑧             ⑨           ⑩    ⑪
```

### 3.2 技术栈

| 组件       | 技术选型                    | 用途            |
| ---------- | --------------------------- | --------------- |
| 向量数据库 | Milvus                      | 存储和检索向量  |
| 应用框架   | LangChain                   | 构建 RAG 流水线 |
| 嵌入模型   | DashScope text-embedding-v1 | 文本向量化      |
| 大语言模型 | Qwen-Plus                   | 生成回答        |
| Web 框架   | Flask                       | 提供 REST API   |
| 文档处理   | PyPDF、Docx2txt 等          | 解析各类文档    |

1. ## 核心概念详解

![img](https://dqej47nflyz.feishu.cn/space/api/box/stream/download/asynccode/?code=MTFjNTI2Zjg2NDJkNzc4ZGE1NTZiMTU4YmI2YWRiYzZfTklSb0d6YlZuY1RPOUlIRHZCcjZOSHlscHh1azVoa1JfVG9rZW46QjV1V2IyNERrbzhWVEN4cXhqamN5a1Y0bklmXzE3ODU4MjUwMzM6MTc4NTgyODYzM19WNA&add_watermark=true&scene_type=CCM)

### 4.1 文本切分（Text Splitting）

为什么需要切分文本？

1. **嵌入模型有长度限制**：大多数模型最多处理 512-8192 个 token
2. **检索精度**：较短的文本块更容易精确匹配
3. **上下文控制**：LLM 的上下文窗口有限

本项目使用 `RecursiveCharacterTextSplitter`：

```Python
self.text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,           # 每块最大 500 字符
    chunk_overlap=50,         # 相邻块重叠 50 字符
    separators=["\n\n", "\n", "。", "！", "？", "；", "，", " ", ""]
)
```

### 4.2 向量嵌入（Embedding）

嵌入是将文本转换为数值向量的过程：

```Plain
"Milvus是一个向量数据库" 
        ↓
    Embedding Model
        ↓
[0.023, -0.156, 0.892, ..., 0.445]  # 1536维向量
```

本项目使用阿里云 DashScope 的 `text-embedding-v1` 模型，输出 1536 维向量。

### 4.3 相似度搜索（Similarity Search）

向量相似度通常使用以下方法计算：

| 方法       | 公式       | 特点         |
| ---------- | ---------- | ------------ |
| L2 距离    | √Σ(aᵢ-bᵢ)² | 值越小越相似 |
| 余弦相似度 | cos(θ)     | 值越大越相似 |
| 内积       | Σ(aᵢ×bᵢ)   | 值越大越相似 |

本项目使用 **L2 距离**（欧氏距离）。

1. ## 项目文件结构

```Plain
project/
├── vector_db_manager.py    # 核心：向量数据库管理器
├── document_loader.py      # 文档加载器（支持多格式）
├── vector_retriever.py     # 向量检索器 + 问答
├── query_system.py         # 独立查询系统（命令行）
├── upload_document.py      # 独立上传脚本（命令行）
├── api_integration.py      # Flask API 蓝图
├── server.py              # Flask 服务器入口
└── .env                   # 环境变量配置
```

### 文件职责说明

| 文件                                                | 主要类/函数           | 职责                            |
| --------------------------------------------------- | --------------------- | ------------------------------- |
| [vector_db_manager.py](http://vector_db_manager.py) | VectorDatabaseManager | 连接 Milvus、文档处理、向量存储 |
| [document_loader.py](http://document_loader.py)     | DocumentLoader        | 加载 PDF/TXT/DOCX/CSV/Excel     |
| [vector_retriever.py](http://vector_retriever.py)   | VectorRetriever       | 封装搜索逻辑、生成回答          |
| [query_system.py](http://query_system.py)           | SimpleQuerySystem     | 独立的 RAG 查询实现             |
| [api_integration.py](http://api_integration.py)     | Flask Blueprint       | REST API 端点定义               |

1. ## 关键代码讲解

### 6.1 向量数据库管理器 (vector_db_manager.py)

这是项目的核心模块，负责整个 RAG 的"写入"流程。

#### 初始化连接

```Python
class VectorDatabaseManager:
    def __init__(self, milvus_host, milvus_port, ...):
        # 1. 配置参数
        self.milvus_host = milvus_host or os.getenv("MILVUS_HOST", "127.0.0.1")
        self.milvus_port = str(milvus_port or os.getenv("MILVUS_PORT", "19530"))
        
        # 2. 初始化嵌入模型
        self._init_embeddings()
        
        # 3. 初始化文本切分器
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=500,
            chunk_overlap=50,
            separators=["\n\n", "\n", "。", "！", "？", "；", "，", " ", ""]
        )
        
        # 4. 连接到 Milvus
        self._connect_to_milvus()
```

#### 文档处理流水线

```Python
def process_file(self, file_path: str, collection_name: str = None) -> bool:
    """处理单个文件：加载 → 切分 → 存储"""
    
    # 步骤1: 加载文档
    documents = self.load_document(file_path)
    # 例如: PDF 文件会被解析为多个 Document 对象
    
    # 步骤2: 切分文档
    split_docs = self.split_documents(documents)
    # 将长文档切分为多个小块
    
    # 步骤3: 存入向量数据库
    self.add_documents_to_db(split_docs, collection_name)
    # 自动调用嵌入模型，生成向量并存储
    
    return True
```

#### 向量存储逻辑

```Python
def add_documents_to_db(self, documents: List[Document], collection_name: str = None):
    """将文档添加到 Milvus"""
    
    target_collection = collection_name or self.collection_name
    collection_exists = utility.has_collection(target_collection)
    
    if collection_exists:
        # 集合已存在：加载并追加数据
        self.vectorstore = Milvus(
            embedding_function=self.embeddings,
            collection_name=target_collection,
            connection_args={"host": self.milvus_host, "port": self.milvus_port}
        )
        self.vectorstore.add_documents(documents)  # 追加
    else:
        # 集合不存在：创建新集合并插入
        self.vectorstore = Milvus.from_documents(
            documents=documents,
            embedding=self.embeddings,
            collection_name=target_collection,
            connection_args={"host": self.milvus_host, "port": self.milvus_port}
        )
```

### 6.2 向量检索器 (vector_retriever.py)

这个模块负责 RAG 的"读取"流程。

#### 相似度搜索

```Python
def search_similar_content(self, query: str, collection_name: str, k: int = None):
    """搜索相似内容"""
    
    # 执行向量搜索（内部会自动将 query 转为向量）
    search_results = self.db_manager.search(
        query=query, 
        k=k, 
        collection_name=collection_name
    )
    
    # 过滤低相似度结果
    results = []
    for doc, score in search_results:
        if score >= self.similarity_threshold:  # 默认 0.5
            results.append((doc, score))
    
    return results
```

#### 问答生成

```Python
def answer_question(self, question: str, collection_name: str, k: int = 5):
    """回答问题 - RAG 的核心逻辑"""
    
    # 步骤1: 检索相关文档
    relevant_docs = self.search_similar_content(
        query=question,
        collection_name=collection_name,
        k=k
    )
    
    # 步骤2: 构建上下文
    context_parts = []
    for i, (doc, score) in enumerate(relevant_docs):
        context_parts.append(f"参考资料{i+1}: {doc.page_content}")
    context = "\n\n".join(context_parts)
    
    # 步骤3: 调用 LLM 生成回答
    answer = self._generate_answer_with_llm(question, context)
    
    return AnswerResult(
        answer=answer,
        source_documents=source_documents,
        scores=scores
    )
```

#### LLM 调用

```Python
def _generate_answer_with_llm(self, question: str, context: str) -> str:
    """使用 LLM 生成回答"""
    
    client = OpenAI(
        api_key=os.environ.get("DASHSCOPE_API_KEY"),
        base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
    )
    
    system_prompt = """你是一个智能助手。请基于提供的【参考资料】回答用户的问题。
    如果参考资料为空或与问题无关，请利用你的通用知识进行回答..."""
    
    response = client.chat.completions.create(
        model="qwen-plus",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"问题：{question}\n\n【参考资料】：\n{context}"}
        ]
    )
    
    return response.choices[0].message.content
```

### 6.3 API 接口 (api_integration.py)

提供 REST API 供外部调用。

#### 上传文档端点

```Python
@vector_bp.route('/upload_document', methods=['POST'])
def upload_document():
    """POST /api/vector/upload_document"""
    
    data = request.get_json()
    file_path = data['file_path']
    collection_name = data['collection_name']
    
    # 调用核心处理逻辑
    success = vector_manager.process_file(file_path, collection_name)
    
    if success:
        return jsonify({
            'success': True,
            'message': f'文档处理成功: {file_path}',
            'database_info': vector_manager.get_database_info(collection_name)
        })
```

#### 查询端点

```Python
@vector_bp.route('/query', methods=['POST'])
def query_documents():
    """POST /api/vector/query"""
    
    data = request.get_json()
    question = data['question']
    collection_name = data['collection_name']
    k = data.get('k', 5)
    
    # 执行 RAG 问答
    result = vector_retriever.answer_question(
        question, 
        k=k, 
        collection_name=collection_name
    )
    
    return jsonify({
        'success': True,
        'question': question,
        'answer': result.answer,
        'confidence': result.confidence,
        'sources': [...]  # 来源文档信息
    })
```

1. ## 快速上手指南

### 7.1 环境准备

1. **启动 Milvus（使用 Docker）**

```Bash
# 创建 docker-compose.yml 并启动
docker-compose up -d
```

1. **安装 Python 依赖**

```Bash
pip install langchain langchain-community langchain-milvus
pip install pymilvus dashscope openai
pip install flask flask-cors python-dotenv
pip install pypdf docx2txt pandas openpyxl
```

1. **配置环境变量**

创建 `.env` 文件：

```Plain
MILVUS_HOST=localhost
MILVUS_PORT=19530
COLLECTION_NAME=agent_rag
DASHSCOPE_API_KEY=your_api_key_here
EMBEDDING_MODEL=text-embedding-v1
LLM_MODEL=qwen-plus
```

### 7.2 命令行使用

**上传文档：**

```Python
# 修改 upload_document.py 中的 FILE_PATH
python upload_document.py
```

**查询问答：**

```Python
# 修改 query_system.py 中的 QUESTION
python query_system.py
```

### 7.3 API 使用

**启动服务：**

```Bash
python server.py
```

**上传文档：**

```Bash
curl -X POST http://localhost:5000/api/vector/upload_document \
  -H "Content-Type: application/json" \
  -d '{"file_path": "/path/to/doc.pdf", "collection_name": "my_docs"}'
```

**提问：**

```Bash
curl -X POST http://localhost:5000/api/vector/query \
  -H "Content-Type: application/json" \
  -d '{"question": "什么是RAG？", "collection_name": "my_docs", "k": 5}'
```

1. ## API 接口说明

| 端点                         | 方法 | 描述                       |
| ---------------------------- | ---- | -------------------------- |
| /api/vector/upload_document  | POST | 上传并处理文档（路径方式） |
| /api/vector/upload_file      | POST | 上传文件流                 |
| /api/vector/query            | POST | RAG 问答                   |
| /api/vector/search           | POST | 纯相似度搜索               |
| /api/vector/collection_info  | GET  | 获取集合信息               |
| /api/vector/clear_collection | POST | 清空集合                   |
