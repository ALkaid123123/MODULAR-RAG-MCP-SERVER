---
name: rag-retrieval
description: "Implement or debug the Retrieval pipeline of the Modular RAG MCP Server: query processing (keyword extraction, filters), dense retriever, sparse retriever (BM25), RRF fusion, reranker (none/cross-encoder/LLM), hybrid search orchestration, or fallback behavior. Use this skill when working on query.py, hybrid search, rerank, retrieval traces, or any code that fetches Top-K chunks from the knowledge base. Align with DEV_SPEC section 3.1.2 and phase D."
---

# RAG Retrieval — 开发指引

本 Skill 用于实现或排查 **检索流水线**（阶段 D）。设计以 **DEV_SPEC.md** 第 3.1.2 节及「阶段 D」任务表为准。

## 何时使用

- 实现/修改 Query 预处理（关键词提取、停用词、filters 解析）
- 实现/修改 DenseRetriever（VectorStore.query、top_k、metadata filter）
- 实现/修改 SparseRetriever（BM25 查询、倒排索引）
- 实现/修改 RRF 融合（k 参数、加权、确定性）
- 实现/修改 HybridSearch 编排（并行两路、结果融合、元数据过滤）
- 实现/修改 Reranker（CoreReranker、None/CrossEncoder/LLM、超时与 Fallback）
- 编写或修复与 retrieval 相关的单元/集成测试

## 检索阶段（多阶段过滤）

1. **Query Processing** — 输入独立查询；输出关键词、filters（collection/doc_type 等）；若需 query 改写/扩展见 DEV_SPEC（默认 dense 单次、sparse 可扩展关键词）。
2. **Dense Retrieval** — Query embedding → 向量库相似度检索 → Top-N 候选。
3. **Sparse Retrieval** — BM25 检索倒排索引 → Top-N 候选。
4. **Fusion** — RRF（Reciprocal Rank Fusion）合并两路排名；公式与 k 见 DEV_SPEC。
5. **Rerank** — 对融合后 Top-M 做精排（None / Cross-Encoder / LLM）；超时或失败时回退到 RRF 排序；精排可选关闭。

Filter 策略：能在检索层做的硬约束做 pre-filter；否则在 fusion/rerank 前做 post-filter；软偏好仅作排序信号（DEV_SPEC 3.1.2）。

## 关键路径与类型

- **Query 输入/输出**：ProcessedQuery、RetrievalResult 等类型见 `src/core/` 或检索模块；与 DEV_SPEC 约定一致。
- **QueryProcessor**：关键词提取、filters；可能依赖 NLP/停用词（见项目实现）。
- **DenseRetriever**：调用 `BaseVectorStore.query`；注入 Embedding 与 VectorStore（工厂从 settings 创建）。
- **SparseRetriever**：调用 BM25Indexer 的查询接口；与 Ingestion 阶段写入的索引一致。
- **RRF**：独立类或函数，输入两路 (id, score/rank)，输出统一排名；k 可配置。
- **HybridSearch**：编排 Dense + Sparse 并行、RRF、可选 Rerank；支持 metadata filter 与 fallback。
- **Reranker**：CoreReranker 调用 Libs 的 RerankerFactory（none/cross_encoder/llm）；实现超时与回退到「无重排」。
- **配置**：`config/settings.yaml` 下 `retrieval`（dense_top_k、sparse_top_k、fusion_top_k、rrf_k）、`rerank`（enabled、provider、top_k 等）。

## 验收与测试

- 阶段 D 每项在 DEV_SPEC 第 6 节「阶段 D：Retrieval MVP」有编号（D1–D7）及验收标准。
- 单元测试：Mock VectorStore/BM25/Reranker；集成测试可用真实 Chroma + 测试数据。
- 脚本入口：`scripts/query.py`（或 `query.py`）支持传入 query、top_k、collection 等，并可选 verbose 输出。

实现或修 bug 时，先确认目标对应的 D 任务与验收标准，再改代码并跑相关测试。
