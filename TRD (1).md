# Medulla TRD

## Architecture

Repository
    ↓
OBSERVE
    ↓
UNDERSTAND
    ↓
REASON
    ↓
ACT

## Layer 1 – Observe
- File watcher
- Git observer
- IDE events
- CLI events

## Layer 2 – Understand
- AST parser
- Repository scanner
- Knowledge graph
- Embedding engine
- Documentation engine

Outputs:
- Architecture
- Features
- APIs
- Business rules
- Dependencies

## Layer 3 – Reason
- Technical debt detection
- Architecture drift
- Duplicate logic
- Complexity analysis
- Missing tests
- Security observations
- Performance observations
- Refactoring opportunities

## Layer 4 – Act
- Documentation generation
- AI context export
- PR summaries
- Daily engineering brief
- Architecture diagrams
- Engineering recommendations


## AI Context and Grounding

### Embedding Generation
- The scanner normalizes repository artifacts into indexable chunks, including source files, symbols, markdown documentation, configuration files, API definitions, dependency manifests, generated docs, and knowledge graph node summaries.
- Each chunk is tagged with repository ID, commit SHA, file path, line range when available, language, symbol name, artifact type, and freshness metadata.
- OpenAI Embeddings generate vector representations for each chunk after parsing and normalization. Embeddings are stored in PostgreSQL with pgvector alongside metadata needed for filtering, citations, and invalidation.
- Incremental updates re-embed only changed chunks and tombstone deleted artifacts so retrieval does not ground answers in stale repository state.

### Retrieval Selection
- Repository Q&A begins with query understanding: classify the request intent, extract referenced files, symbols, features, technologies, and time-sensitive constraints.
- Retrieval combines vector similarity, keyword search, metadata filters, and graph traversal. The system should select relevant files, symbols, docs, and graph nodes rather than relying on embeddings alone.
- File and symbol matches are prioritized when the user names concrete artifacts. Documentation and generated summaries are used to provide architectural or product context. Knowledge graph nodes and edges are used to include relationships such as imports, ownership, API calls, feature mappings, and dependencies.
- Results are ranked by relevance, freshness, artifact authority, and graph proximity, then deduplicated and compacted into source-grounded context blocks with stable references.

### Prompt Assembly for Repository Q&A
- Prompts include the user question, repository identity, current commit or indexed revision, retrieved context blocks, source references, and any known limitations of the index.
- Context blocks must preserve provenance: file path, line range when available, symbol name, artifact type, and graph node ID when applicable.
- The prompt should instruct the model to answer only from supplied repository context unless it explicitly labels an item as an inference or general engineering recommendation.
- For multi-step questions, prompts may include a concise reasoning plan, but final answers should emphasize evidence, citations, and actionable conclusions rather than hidden chain-of-thought.

### Referencing Source Artifacts
- Generated docs and recommendations should reference the source artifacts that support them, including file paths, symbols, docs, graph nodes, issues, pull requests, or prior generated reports.
- Recommendations should separate evidence from proposed action: cite what the repository currently contains, then explain the inferred risk, tradeoff, or improvement opportunity.
- PR summaries and engineering briefs should link changes or observations back to modified files, relevant symbols, tests, dependency updates, and impacted architectural areas.

### Insufficient Context Handling
- If retrieval returns weak, conflicting, or insufficient evidence, the system should say that context is insufficient before answering.
- The response should list what was checked, what is missing, and the safest next step, such as rescanning, adding documentation, opening a specific file, or asking the user for clarification.
- The system must not fabricate file contents, line numbers, dependencies, ownership, or implementation details that are not present in indexed context.

### Required Response Shapes
- **Chat answers:** concise answer first, followed by supporting repository references, known facts, inferred recommendations, and any context gaps.
- **PR summaries:** summary of user-facing and technical changes, affected files or symbols, tests or checks observed, risks, and follow-up recommendations with source references.
- **Engineering briefs:** executive summary, current repository signals, notable changes, risks, opportunities, recommended actions, and cited source artifacts.
- **Recommendations:** problem statement, evidence, inferred impact, confidence level, suggested action, estimated effort when possible, and links or references to the supporting artifacts.

## Core Components
- FastAPI backend
- PostgreSQL + pgvector
- Async SQLAlchemy
- watchfiles
- OpenAI Responses API
- OpenAI Embeddings
- Next.js dashboard
- WebSockets

## CLI
medulla init
medulla scan
medulla status
medulla ask
medulla export
medulla doctor

## Dashboard
- Repository Health
- Architecture
- Features
- Knowledge
- Timeline
- Insights
- Recommendations
- AI Chat

## Performance Targets
- Initial scan <60s
- Incremental update <2s
- Search <300ms
- Insight refresh <5s

## Future
Cloud sync, VS Code extension, GitHub integration, organization knowledge graph, MCP server.
