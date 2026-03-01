---
name: modular-rag-dev
description: "Work on the Modular RAG MCP Server codebase: implement features from DEV_SPEC, add or replace providers (LLM/Embedding/VectorStore/Splitter/Reranker), follow project conventions (factory pattern, config/settings.yaml, core types), run or fix tests (pytest, unit/integration), or extend the RAG pipeline. Use this skill whenever the user is developing, debugging, or refactoring in this repository, mentions DEV_SPEC or task IDs (A1, B7, C4, D5, etc.), or asks how to add a new provider or module. Prefer consulting DEV_SPEC.md for acceptance criteria and architecture."
---

# Modular RAG MCP Server — 开发规范

在为本项目编写或修改代码时，请严格对齐 **DEV_SPEC.md** 的架构、目录与验收标准。本 Skill 提供开发过程中的约定与入口索引。

## 何时使用本 Skill

- 实现或修改任意模块（Ingestion / Retrieval / MCP / Trace / Dashboard / Evaluation）
- 新增或替换可插拔实现（新 LLM/Embedding/VectorStore/Splitter/Reranker/Evaluator）
- 查找任务验收标准、测试方式或配置项
- 运行/修复单元测试或集成测试（pytest）
- 理解项目目录、数据契约（Document/Chunk/ChunkRecord）或工厂/配置驱动方式

## 项目根与权威文档

- **项目根**：仓库根目录（含 `DEV_SPEC.md`、`config/settings.yaml`、`src/`、`tests/`）。
- **权威规范**：所有设计、排期、验收标准以 **DEV_SPEC.md** 为准；实现前请先查阅对应章节。

## 目录与分层（对齐 DEV_SPEC 5.2）

```
src/
├── core/           # 配置、类型、Trace 基础设施
├── ingestion/      # Pipeline、Loader、Chunking、Transform、Embed、Upsert、DocumentManager
├── mcp_server/     # MCP Server、Tools、Protocol
├── libs/           # 可插拔层：llm, embedding, splitter, vector_store, reranker, evaluator
└── observability/  # 日志、Dashboard、评估
config/
├── settings.yaml   # 唯一配置入口，驱动所有 Factory
└── prompts/        # Rerank、Metadata Enrichment、Image Captioning 等模板
tests/
├── unit/
└── integration/
```

- **Libs**：仅提供抽象接口 + 工厂 + 默认实现；不依赖 `ingestion` 或 `mcp_server`。
- **Core**：`Settings`、`Document`/`Chunk`/`ChunkRecord`、`TraceContext` 等契约与工具。
- **配置驱动**：通过 `settings.yaml` 的 `provider`/`backend` 等字段切换实现，业务代码不写死具体类。

## 开发约定

1. **先看 DEV_SPEC**：每个任务在 DEV_SPEC 中都有编号（如 C4、D5、E3）和验收标准；实现后需满足对应测试或验收方法。
2. **TDD 优先**：新增逻辑先写/补单元测试（`tests/unit/`），外部依赖用 Fake/Mock；集成测试可选真实后端。
3. **工厂 + 抽象**：新增一种 Provider 时，实现对应 Base 接口并在对应 Factory 中注册，在 `config/settings.yaml` 中增加可选配置。
4. **类型与契约**：文档/块/记录等统一使用 `src/core/types.py`（或 DEV_SPEC 规定的类型）；不要在各处重复定义等价结构。
5. **日志与 Trace**：需要可观测时使用 `TraceContext` 与现有 logger（见可观测性设计），stdout 仅用于 MCP 协议输出，日志走 stderr/文件。

## DEV_SPEC 章节索引（快速定位）

| 章节 | 内容 |
|------|------|
| 1–2 | 项目概述、核心特点、设计理念 |
| 3.1 | 数据摄取流水线（Loader/Splitter/Transform/Embed/Upsert）、文档生命周期 |
| 3.1.2 | 检索流水线（Query Processing、Hybrid、RRF、Rerank） |
| 3.2 | MCP 服务设计（Stdio、Tools、返回与引用） |
| 3.3 | 可插拔架构（LLM/Embedding/VectorStore/Splitter/检索/评估）、配置管理 |
| 3.4 | 可观测性（Trace 数据结构、TraceContext、双链路打点） |
| 3.5 | 多模态（Image Captioning、存储与检索） |
| 4 | 测试方案（单元/集成/E2E、覆盖率） |
| 5 | 系统架构与模块设计、目录树 |
| 6 | **项目排期**：阶段 A–I、任务表、进度、验收标准 |

实现某阶段时，请直接打开 **DEV_SPEC.md** 对应小节（如「阶段 C：Ingestion Pipeline」「阶段 D：Retrieval」）按任务编号逐项完成。

## 常用命令

```bash
# 运行全部单元测试
pytest tests/unit -v

# 运行指定阶段相关测试（示例）
pytest tests/unit -v -k "config or settings"
pytest tests/unit -v -k "llm or embedding"
pytest tests/integration -v

# 加载配置（Python）
from src.core.settings import load_settings
settings = load_settings()
```

## 扩展新 Provider 的通用步骤

1. 在 `src/libs/<domain>/` 下实现对应 `Base*` 接口（如 `BaseLLM`、`BaseEmbedding`）。
2. 在对应 `*_factory.py` 中注册（如 `LLMFactory.register_provider("my_provider", MyLLM)`）。
3. 在 `config/settings.yaml` 中增加该 provider 所需配置项（若与现有共用则复用已有 key）。
4. 为新区块编写单元测试（Fake 或 Mock 外部 API），必要时补集成测试。

完成以上后，在排期表中更新对应任务状态并注明完成日期（见 DEV_SPEC 第 6 节进度表）。
