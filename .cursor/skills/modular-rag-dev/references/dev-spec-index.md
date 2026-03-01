# DEV_SPEC 章节索引

便于在实现时快速定位 **DEV_SPEC.md** 中的小节。以仓库内 DEV_SPEC.md 为准，此处仅作索引。

| 章节 | 主题 |
|------|------|
| 1 | 项目概述 |
| 2 | 核心特点（RAG 策略、可插拔、MCP、多模态、可观测、扩展性） |
| 3.1 | 数据摄取流水线（Loader/Splitter/Transform/Embed/Upsert、文档生命周期、DocumentManager） |
| 3.1.2 | 检索流水线（Query Processing、Hybrid、RRF、Rerank、Filter） |
| 3.2 | MCP 服务设计（Stdio、Tools、返回与引用、多模态） |
| 3.3 | 可插拔架构（LLM/Embedding/VectorStore/Splitter/检索/评估、配置管理） |
| 3.4 | 可观测性（Trace 数据结构、TraceContext、双链路、Dashboard 概要） |
| 3.5 | 多模态（Image Captioning、Loader/Splitter/Transform/Storage 各阶段要点） |
| 4 | 测试方案 |
| 5 | 系统架构与模块设计、目录树 |
| 6 | **项目排期**：阶段 A–I、任务表、进度、验收标准 |

## 阶段与任务编号

- **A**：工程骨架与测试基座（A1–A3）
- **B**：Libs 可插拔层（B1–B9）
- **C**：Ingestion Pipeline MVP（C1–C15）
- **D**：Retrieval MVP（D1–D7）
- **E**：MCP Server 与 Tools（E1–E6）
- **F**：Trace 基础设施（F1–F5）
- **G**：Dashboard（G1–G6）
- **H**：评估体系（H1–H5）
- **I**：E2E 与文档收口（I1–I5）
