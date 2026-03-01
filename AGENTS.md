# AGENTS

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:
- Invoke: `npx openskills read <skill-name>` (run in your shell)
  - For multiple: `npx openskills read skill-one,skill-two`
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>frontend-design</name>
<description>Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, or applications. Generates creative, polished code that avoids generic AI aesthetics.</description>
<location>project</location>
</skill>

<skill>
<name>pptx</name>
<description>"Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions \"deck,\" \"slides,\" \"presentation,\" or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill."</description>
<location>project</location>
</skill>

<skill>
<name>skill-creator</name>
<description>Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, update or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.</description>
<location>project</location>
</skill>

<skill>
<name>modular-rag-dev</name>
<description>Work on the Modular RAG MCP Server codebase: implement features from DEV_SPEC, add or replace providers (LLM/Embedding/VectorStore/Splitter/Reranker), follow project conventions (factory pattern, config/settings.yaml, core types), run or fix tests (pytest, unit/integration), or extend the RAG pipeline. Use when developing, debugging, or refactoring in this repo, or when mentioning DEV_SPEC or task IDs (A1, B7, C4, D5, etc.).</description>
<location>project</location>
</skill>

<skill>
<name>rag-ingestion</name>
<description>Implement or debug the Ingestion Pipeline: PDF/MD loading, chunking, transform (ChunkRefiner, MetadataEnricher, ImageCaptioner), embedding, BM25, vector upsert, image storage, pipeline orchestration, DocumentManager, file integrity. Use when working on ingest.py, document chunking, or ingestion traces. Align with DEV_SPEC 3.1 and phase C.</description>
<location>project</location>
</skill>

<skill>
<name>rag-retrieval</name>
<description>Implement or debug the Retrieval pipeline: query processing, dense/sparse retriever, RRF fusion, reranker, hybrid search orchestration, fallback. Use when working on query.py, hybrid search, rerank, or retrieval traces. Align with DEV_SPEC 3.1.2 and phase D.</description>
<location>project</location>
</skill>

<skill>
<name>mcp-server-dev</name>
<description>Implement or modify the MCP Server and tools: stdio transport, protocol handling, query_knowledge_hub, list_collections, get_document_summary, response format (citations, multimodal). Use when working on server entrypoint, tool handlers, or MCP protocol code. Align with DEV_SPEC 3.2 and phase E.</description>
<location>project</location>
</skill>

<skill>
<name>rag-observability</name>
<description>Implement or debug observability: TraceContext, query/ingestion traces, stage recording, JSON Lines logging, on_progress callback, Streamlit Dashboard (overview, data browser, ingestion/query traces, evaluation panel). Use when working on tracing, logs, or dashboard. Align with DEV_SPEC 3.4 and phases F–G.</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>
