---
name: mcp-server-dev
description: "Implement or modify the MCP Server and tools of the Modular RAG MCP Server: stdio transport, protocol handling, tool definitions (query_knowledge_hub, list_collections, get_document_summary), response format (citations, multimodal text+image), or MCP SDK usage. Use this skill when working on server entrypoint, tool handlers, response builders, citation generation, or any MCP protocol/capability code. Align with DEV_SPEC section 3.2 and phase E."
---

# MCP Server 与 Tools — 开发指引

本 Skill 用于实现或排查 **MCP 服务层与 Tools**（阶段 E）。设计以 **DEV_SPEC.md** 第 3.2 节及「阶段 E」任务表为准。

## 何时使用

- 实现/修改 MCP Server 入口（Stdio、进程生命周期）
- 实现/修改协议处理（能力协商、tool 注册、错误映射）
- 实现/修改 Tools：`query_knowledge_hub`、`list_collections`、`get_document_summary`
- 实现/修改返回格式（结构化引用、Markdown、多模态 Text + Image）
- 实现/修改 ResponseBuilder、CitationGenerator、MultimodalAssembler 等
- 编写或修复与 MCP 相关的单元/集成测试

## 设计要点（DEV_SPEC 3.2）

- **传输**：仅 Stdio；stdout 仅输出合法 MCP 消息，日志走 stderr。
- **SDK**：优先使用 Python 官方 MCP SDK（`mcp`），声明式注册 tools。
- **Tools**：
  - `query_knowledge_hub`：主检索入口；参数如 query、top_k、collection；返回带引用的结构化结果。
  - `list_collections`：列举集合名称、描述、文档数等。
  - `get_document_summary`：按 doc_id 返回文档摘要与元信息。
- **引用透明**：每个检索片段含 source_file、page、chunk_id、score；推荐 structuredContent 中统一 Citation 格式；content 数组首项为 Markdown 可读回答（含 [1] 等引用标注）。
- **多模态**：检索命中关联图片时，Server 读本地文件并 Base64 编码返回；格式见 DEV_SPEC（type/image、mimeType）；首项仍为文本以保证兼容性。

## 关键路径

- **Server 入口**：`src/mcp_server/` 下 server 启动与 Stdio 绑定（具体文件名见项目）。
- **Protocol / Tool 注册**：ProtocolHandler 或等价层；tool 列表与参数 schema 符合 MCP 规范。
- **Tool 实现**：各 tool 调用 Core/Ingestion 层（HybridSearch、DocumentManager 等），组装 MCP 规定的 content 与 structuredContent。
- **ResponseBuilder / CitationGenerator / MultimodalAssembler**：见阶段 E 任务描述与现有实现。
- **配置**：无单独 MCP 配置块时，复用 `config/settings.yaml` 中 LLM、VectorStore、Retrieval 等。

## 验收与测试

- 阶段 E 每项在 DEV_SPEC 第 6 节「阶段 E：MCP Server 层与 Tools」有编号（E1–E6）及验收标准。
- 单元测试：Mock HybridSearch、DocumentManager；集成测试可启动 Server 并通过 Stdio 发送 JSON-RPC 请求验证 tools。

实现或修 bug 时，先确认目标对应的 E 任务与验收标准，再改代码并跑相关测试。
