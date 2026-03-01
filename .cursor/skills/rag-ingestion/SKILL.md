---
name: rag-ingestion
description: "Implement or debug the Ingestion Pipeline of the Modular RAG MCP Server: PDF/MD loading, chunking (splitter), transform (ChunkRefiner, MetadataEnricher, ImageCaptioner), embedding (dense/sparse), BM25 index, vector upsert, image storage, pipeline orchestration, DocumentManager, or file integrity (SHA256/skip). Use this skill when working on ingest.py, document chunking, chunk refinement, metadata enrichment, image captioning, batch embed/upsert, or ingestion traces. Always align with DEV_SPEC section 3.1 and phase C task list."
---

# RAG Ingestion Pipeline — 开发指引

本 Skill 用于实现或排查 **数据摄取流水线**（阶段 C）。所有设计与验收标准以 **DEV_SPEC.md** 第 3.1 节及「阶段 C」任务表为准。

## 何时使用

- 实现/修改 Loader（PDF→Markdown、元数据抽取、图片引用）
- 实现/修改分块（DocumentChunker、Splitter 集成、chunk 元数据）
- 实现/修改 Transform（ChunkRefiner、MetadataEnricher、ImageCaptioner）
- 实现/修改编码与存储（DenseEncoder、SparseEncoder、BM25Indexer、VectorUpserter、ImageStorage）
- 编排 Pipeline（串行阶段、on_progress 回调、异常与增量）
- 实现/使用 DocumentManager（list_documents、get_document_detail、delete_document、跨存储协调）
- 文件完整性检查（SHA256、ingestion_history、跳过未变更文件）
- 编写或修复与 ingestion 相关的单元/集成测试

## 流水线阶段（顺序固定）

1. **Load** — Loader 将源文件解析为统一 `Document`（text 为规范化 Markdown + metadata）。
2. **Split** — Splitter（如 RecursiveCharacterTextSplitter）将 Document 切为 Chunk，保留 source/chunk_index 等定位信息。
3. **Transform** — 可选链：ChunkRefiner → MetadataEnricher → ImageCaptioner；输出增强后的 Chunk。
4. **Embed** — DenseEncoder + SparseEncoder 批量生成向量；差量/内容哈希优化见 DEV_SPEC。
5. **Upsert** — VectorUpserter 写向量库 + BM25 索引；ImageStorage 存图片并写 SQLite 索引；幂等以 chunk_id 为准。

Pipeline 应支持 `on_progress(stage_name, current, total)` 回调供 Dashboard 展示进度；各阶段完成后通过 `TraceContext.record_stage()` 打点（见可观测性设计）。

## 关键路径与类型

- **核心类型**：`src/core/types.py` — Document, Chunk, ChunkRecord；与 DEV_SPEC 中数据契约一致。
- **Pipeline 编排**：`src/ingestion/pipeline.py`（或等价入口）串起 load → split → transform → embed → upsert。
- **Loader**：`src/ingestion/loader/`，实现 `BaseLoader`；当前至少 PDF→Markdown（如 MarkItDown）。
- **Chunking**：`src/ingestion/chunking/`，调用 Libs 的 SplitterFactory，产出带 metadata 的 Chunk。
- **Transform**：`src/ingestion/transform/` — ChunkRefiner、MetadataEnricher、ImageCaptioner；均实现 BaseTransform，可配置 rule-based 或 LLM。
- **Embed / Upsert**：DenseEncoder、SparseEncoder、BM25Indexer、VectorUpserter、ImageStorage 的模块位置见 DEV_SPEC 5.2 目录树。
- **DocumentManager**：跨 Chroma、BM25、ImageStorage、FileIntegrity 的协调删除与列表；见 DEV_SPEC 3.1 文档生命周期。
- **配置**：`config/settings.yaml` 下 `ingestion`（chunk_size、chunk_overlap、splitter、batch_size、chunk_refiner、metadata_enricher 等）；prompts 在 `config/prompts/`。

## 验收与测试

- 阶段 C 每项任务在 DEV_SPEC 第 6 节「阶段 C：Ingestion Pipeline MVP」中有对应编号（C1–C15）及完成标准。
- 单元测试：Fake/Mock LLM 与 Embedding；集成测试可选用真实后端。
- 脚本入口：`scripts/ingest.py`（或 `ingest.py`）支持指定路径/集合、增量跳过、错误处理；E2E 可覆盖「单文件 ingest → 查询可见」。

实现或修 bug 时，先确认目标对应的 C 任务与验收标准，再改代码并跑相关 `tests/unit` / `tests/integration`。
