---
title: 【RAG】分块大小与重叠窗口调优方案
date: 2026-09-30 17:22:14
tags:
    - RAG
    - 分块

---
# 分块大小与重叠窗口调优方案

项目地址：https://github.com/WiSiW/AI-RAG

## 摘要

新增独立的“分块调优”工作台，以固定评测集和黄金证据为依据，比较分块大小、重叠窗口与检索质量、上下文噪声、答案质量和计算成本之间的关系。

方案采用两阶段评测：

1. **快速筛选**：对默认 `256/512/1024/1536 × 0%/10%/20%/25%` 共 16 组参数，构建隔离的 L2 原文向量和 BM25 索引，计算 Recall、MRR、上下文证据密度、延迟和索引成本。
2. **深度验证**：管理员从 Pareto 前沿中选择最多 5 组候选，系统构建完整 L1/L2/L3 多表示索引，对每个候选分层抽取 30 个可回答问题生成答案，评估忠实度、完整性、引用利用率和完整检索指标。

所有实验使用临时索引命名空间，不修改生产文档、生产 chunk 或生产 Milvus/Whoosh 索引。只有用户确认后，才将结果保存为知识库默认参数；现有文档重建必须另行明确确认。

## 评测集与指标

- 评测集按单个知识库管理，覆盖该库中选择的代表性文档；每条问题记录类型：`detail/comparison/summary/reasoning/table/code/no_answer`。
- 可回答问题必须提供标准答案和至少一条黄金证据；证据通过原文划选产生，保存原文片段、归一化片段、文档、章节、字符位置和可选页码，不绑定现有 `chunk_id`。
- 黄金片段限制为最小充分证据，建议不超过 300 token；多文档比较多证据题允许添加多条。
- 运行至少要求 20 条可回答问题，界面持续提示补足到推荐的 50–100 条；无答案题不进入 Recall 分母，单独统计拒答正确率。
- 证据命中定义为：归一化空白和换行后，黄金片段完整出现在某条召回 chunk 的原文中。
- 核心指标：
  - `Recall@K = 前 K 个 chunk 命中的不重复黄金片段数 / 该题黄金片段总数`，报告 @5、@10。
  - `AllEvidenceHit@K`：全部黄金片段均被前 K 个 chunk 覆盖的问题比例。
  - `MRR@10`：首个黄金证据排名的倒数均值。
  - `ContextEvidenceDensity@K`：命中的不重复黄金 token 数除以全部召回 chunk token 数，作为块内噪声和利用率代理指标。
  - `p50/p95 检索延迟`、候选 chunk 数、索引文本 token 数、估算向量存储字节、构建耗时和失败率。
  - 深度阶段增加 LLM 判定：Faithfulness 1–5、Completeness 1–5；`CitedGoldChunkRatio = 被引用且包含黄金证据的 chunk 数 / 全部被引用 chunk 数`。
- 结果同时提供相对当前生产参数基线的差值和 95% bootstrap 置信区间。Pareto 前沿以 Recall@5、Recall@10、上下文证据密度最大化，p95 延迟和索引 token 成本最小化计算；不合成单一总分。
- 推荐 Top 3 仅作为辅助：优先保留 p95 不超过基线 1.5 倍、索引 token 不超过基线 2 倍的 trials，再按 Recall@5、Recall@10、证据密度、成本排序。

## 后端与执行链路

- 新增评测集表：数据集、问题、黄金证据；数据集编辑后版本递增，历史运行保存数据、原文哈希和配置快照。
- 新增调优运行表：运行、trial、逐题结果。trial 保存实际 chunk token 数、重叠 token 数、阶段、状态、检索/质量/成本指标和原始错误；逐题结果保存排名、命中证据、Top 10 chunk 快照、答案和质量评分。
- 状态使用 `queued/running/awaiting_selection/completed/partial/failed/cancelled`。快速阶段成功后可进入 `awaiting_selection`，深度验证由管理员勾选候选后创建关联运行。
- 每个知识库同一时间只允许一个运行中的调优任务，避免模型配额和索引资源争用。
- 文档只解析一次生成固定解析快照，所有 trial 复用该快照，仅重新分块。保存解析器配置和解析内容哈希；快照不一致时拒绝进入深度验证。
- 快速阶段仅生成 L2 原文表示和 BM25，使用固定的文档/章节前缀、固定 BM25 与 original 权重、`candidate_top_k=20`、`final_top_k=10`、单跳检索且关闭 Rerank。每个问题只生成一次查询向量并复用于所有 trial。
- 深度阶段复用现有增强、QA、L1/L2/L3 构建和 RRF/Rerank 逻辑，固定知识库 Embedding、增强、Rerank、LLM 模型及 Prompt 版本。每个候选额外保留当前参数作为对照 trial。
- 自定义大小和重叠按矩阵展开；重叠百分比使用 `round(chunk_size × ratio)` 转为实际 token，并验证 `0 <= overlap < chunk_size`。默认 16 组，最多 25 组。
- 临时 Milvus collection 和 Whoosh 目录使用 `eval_<trial_id>` 命名空间。每个 trial 在 `finally` 中清理索引；失败 trial 不阻塞其他 trial。运行删除、启动前检查和历史清理都会回收遗留临时索引。
- 深度阶段构建完整性失败或增强降级时标记 `partial` 并记录原因；只有无关键缺失的 `completed` trial 可以应用。
- 应用操作只更新知识库 `settings.chunk_mode/chunk_size/chunk_overlap` 并记录调优运行和 trial 引用，不自动重建文档。页面另提供“按新参数重建现有文档”确认操作，批量更新所选文档并复用现有入库任务状态机。

## API 与页面

- 新增评测集 API：`/retrieval-evaluation-datasets` 的增删改查、问题增删改、JSONL 原子导入和导出。
- 新增调优 API：`POST/GET /chunk-tuning-runs`、运行详情、trials、逐题结果、取消、删除；`POST /chunk-tuning-runs/{id}/verifications` 接收 `trial_ids` 创建深度验证。
- `POST /chunk-tuning-runs/{id}/apply` 接收 `trial_id`，仅保存默认参数；`POST /knowledge-bases/{kb_id}/chunk-settings/reindex` 接收文档列表和目标参数，显式创建重建任务。
- JSONL 带 `schema_version`、知识库、问题、标准答案、问题类型和多条证据；导入前完成归属、证据可定位性、重复问题和必填字段校验，全部通过才提交。
- 所有请求/响应使用 `schemas.py` 类型模型，并同步 `frontend/src/types/index.ts` 与 API client，字段保持 snake_case。
- 新增 `/chunk-tuning` 导航页和四个区域：
  - **评测集**：问题表格、类型筛选、原文划选添加证据、JSONL 导入导出。
  - **快速筛选**：参数矩阵编辑器、当前基线、进度、失败原因和取消。
  - **结果分析**：指标排行榜、基线差值、Pareto 标签、可切换坐标轴的 SVG 散点图、延迟/索引成本对比和逐题召回明细。
  - **深度验证**：勾选候选、30 题答案质量结果、引用证据对照、最终推荐和保存默认参数。
- 不新增图表库，使用现有 Tailwind 风格和原生 SVG；表格、散点图和详情面板复用现有 UI 组件风格。
- 同步更新 `docs/api.md`、`docs/rag-evaluation.md`，并新增一份设计说明，明确指标公式、实验限制、Pareto 规则和应用流程。

## 测试与验收

- 单元测试覆盖矩阵生成、百分比转 token、非法重叠、黄金片段归一化、多证据 Recall、MRR、证据密度和基线差值。
- 测试多文档问题、同一片段重复命中、黄金片段不存在、无答案题排除、LLM 评判失败和 bootstrap 计算。
- 使用 fake gateway/vector store/BM25 store 验证每个问题只 Embedding 一次、快速阶段路由受控、深度阶段模型和 RRF 配置固定。
- 验证临时索引创建、成功/失败/取消后的清理，以及全过程不改变生产 collection、Whoosh 目录、DocumentChunk 和 Document 参数。
- API 测试覆盖权限、知识库归属、单库限制、数据集版本冻结、JSONL 原子导入、并发运行冲突、partial 状态和 apply 限制。
- 前端测试覆盖矩阵编辑、轮询、Pareto 选择、逐题证据查看、保存默认参数和显式重建确认，并通过生产构建。
- 最终运行 `ruff check app tests`、完整 `pytest -q` 和 `npm.cmd run build`。
- 验收目标：固定数据集包含 50–100 条真实分层问题；16 组快速评测均产生可解释结果；深度候选每类问题均有覆盖；选中的参数能明确说明相对基线的 Recall、噪声、延迟和答案质量取舍，且不修改生产索引。

## 假设与默认值

- 分块调优以单个知识库为范围；多知识库联合调优不在首版范围。
- 默认使用 `structure` 分块模式，分块大小和重叠是唯一变化的实验变量；语义分块阈值、解析器、模型、Prompt、RRF 权重和 Top-K 全部固定。
- 快速筛选使用接近生产原文的确定性前缀，但不生成 L1 摘要；其定位是候选筛选，不直接作为最终应用依据。
- 深度验证每候选分层抽取 30 个可回答问题；无答案题单独评估，不计入 Recall。
- 不保留临时向量索引，只长期保存运行指标、Top 10 命中快照和分析结果。
- 生产参数变更必须由管理员显式执行；调优运行、查看结果或删除运行都不会自动重入库。
