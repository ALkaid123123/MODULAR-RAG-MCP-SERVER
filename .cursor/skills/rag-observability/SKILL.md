---
name: rag-observability
description: "Implement or debug observability in the Modular RAG MCP Server: TraceContext, trace types (query/ingestion), stage recording, JSON Lines trace logging, pipeline on_progress callback, or Streamlit Dashboard (overview, data browser, ingestion management, ingestion traces, query traces, evaluation panel). Use this skill when working on tracing, logs, dashboard pages, or trace-driven UI. Align with DEV_SPEC section 3.4 and phases F–G."
---

# RAG 可观测性 — 开发指引

本 Skill 用于实现或排查 **Trace 基础设施与可视化管理平台**（阶段 F、G）。设计以 **DEV_SPEC.md** 第 3.4 节及阶段 F/G 任务表为准。

## 何时使用

- 实现/修改 TraceContext（trace_id、trace_type、record_stage、finish、to_dict、耗时统计）
- 实现/修改结构化日志（JSON Lines、trace 写入、trace_file 配置）
- 在 Query 或 Ingestion 链路中打点（各阶段 record_stage）
- 实现/修改 Pipeline 的 on_progress 回调（stage_name、current、total）
- 实现/修改 Dashboard（Streamlit）：系统总览、数据浏览器、Ingestion 管理、Ingestion 追踪、Query 追踪、评估面板
- 实现/修改 Trace 读取与展示（TraceService、阶段耗时、瀑布图等）
- 编写或修复与 trace/dashboard 相关的单元/集成测试

## 双链路 Trace（DEV_SPEC 3.4）

- **Query Trace**：trace_type=query；阶段包括 query_processing、dense、sparse、fusion、rerank；记录候选数、分数、耗时、最终 top_k 等。
- **Ingestion Trace**：trace_type=ingestion；阶段包括 load、split、transform、embed、upsert；记录 chunk 数、method/provider、耗时等。
- **TraceContext**：请求级/摄取级创建；各组件在执行后调用 `record_stage(name, ...)`；最后 `finish()` 并可通过 `to_dict()` 序列化写入 JSON Lines。
- **低侵入**：业务逻辑与打点解耦；通过参数传入 TraceContext，不污染全局状态。
- **动态展示**：Dashboard 根据 trace 中的 method/provider/details 动态渲染，更换可插拔组件后无需改 Dashboard 代码。

## 关键路径

- **TraceContext**：`src/core/trace/`（或 observability 下）；支持 trace_type、record_stage、elapsed_ms、finish。
- **Trace 日志**：JSON Formatter、get_trace_logger、写入 observability.trace_file（如 ./logs/traces.jsonl）。
- **Query 打点**：HybridSearch、CoreReranker 等在执行各阶段后注入 trace 并 record_stage。
- **Ingestion 打点**：Pipeline 各阶段（load/split/transform/embed/upsert）完成后 record_stage；on_progress 与 trace 可并行存在。
- **Dashboard**：`src/observability/dashboard/`；多页面（overview、data、ingestion、ingestion_traces、query_traces、evaluation_panel）；TraceService 读取 traces.jsonl 并提供按类型/时间筛选。
- **配置**：`config/settings.yaml` 下 observability（log_level、trace_enabled、trace_file、structured_logging）、dashboard（enabled、port 等若存在）。

## 验收与测试

- 阶段 F（Trace 基础设施）与阶段 G（Dashboard）在 DEV_SPEC 第 6 节有对应任务（F1–F5、G1–G6）及验收标准。
- 单元测试：TraceContext 的 record_stage/finish/to_dict；logger 输出格式。
- 集成测试：跑一次 ingestion 与一次 query，检查 traces.jsonl 中是否出现两条 trace 且阶段完整；Dashboard 能打开并展示 trace 列表与详情。

实现或修 bug 时，先确认目标对应的 F/G 任务与验收标准，再改代码并跑相关测试。
