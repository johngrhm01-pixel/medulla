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

## Core Components
- FastAPI backend
- PostgreSQL + pgvector
- Async SQLAlchemy
- watchfiles
- OpenAI Responses API
- OpenAI Embeddings
- Next.js dashboard
- WebSockets

## System Responsibilities

### CLI
- Provides the local developer entrypoint for setup, scans, status checks, questions, exports, and diagnostics.
- Translates commands such as `medulla scan` into backend jobs and reports progress or failures to the user.
- Supplies repository paths, configuration, and authentication context to the backend.

### FastAPI backend
- Exposes HTTP APIs for scan orchestration, repository metadata, search, insights, and dashboard queries.
- Coordinates parsing, persistence, embedding generation, graph updates, and insight refreshes.
- Owns request validation, background task scheduling, service-level error handling, and integration boundaries.

### Repository scanner
- Walks the repository tree during initial or manual scans and selects files that should be analyzed.
- Applies ignore rules, file type filters, size limits, and change detection before handing files to the parser layer.
- Produces a consistent inventory of source files, configuration files, documentation, and dependency manifests.

### File watcher
- Observes repository changes after initialization using filesystem events.
- Debounces rapid edits and converts creates, updates, deletes, and renames into incremental analysis jobs.
- Keeps the stored repository model current without requiring a full rescan.

### AST/parser layer
- Parses supported languages and file formats into structured symbols, relationships, APIs, imports, comments, and metadata.
- Normalizes parser output into repository entities that downstream graph, embedding, and insight services can consume.
- Falls back to lightweight text or metadata extraction when a full AST is unavailable.

### Embedding engine
- Generates vector representations for code, documentation, symbols, and higher-level knowledge artifacts.
- Maintains embeddings when source content changes and removes obsolete vectors when files are deleted.
- Enables semantic search, related-code discovery, AI context retrieval, and insight enrichment.

### Knowledge graph builder
- Converts parsed entities into a graph of files, modules, symbols, APIs, dependencies, features, and business rules.
- Updates relationships incrementally as scans and watcher events discover changes.
- Provides graph queries used for architecture views, dependency tracing, drift detection, and impact analysis.

### Reasoning/insights engine
- Evaluates repository data, graph relationships, embeddings, and historical changes to identify actionable engineering observations.
- Produces findings such as technical debt, duplicated logic, missing tests, security concerns, performance risks, and refactoring opportunities.
- Ranks and refreshes insights so the dashboard and API surface current, explainable recommendations.

### PostgreSQL/pgvector storage
- Persists repository metadata, parsed entities, graph relationships, scan history, events, insights, and user-facing artifacts.
- Stores vector embeddings with pgvector for semantic retrieval alongside relational project data.
- Provides transactional consistency, queryable history, and durable state for the backend and dashboard.

### Next.js dashboard
- Presents repository health, architecture, features, knowledge, timelines, insights, recommendations, and AI chat experiences.
- Consumes backend APIs and WebSocket updates to keep views current during scans and file changes.
- Turns stored analysis into navigable, human-readable engineering intelligence.

### WebSocket event stream
- Pushes scan progress, file change events, parser updates, insight refreshes, and system status changes to connected clients.
- Allows the dashboard and other consumers to react to repository updates without polling.
- Provides a real-time feedback loop from backend processing to developer-facing interfaces.

## Data Flow

1. A developer starts `medulla scan`, or the file watcher detects a create, update, delete, or rename event in an initialized repository.
2. The CLI or watcher sends repository path, configuration, and change context to the FastAPI backend, which schedules a scan or incremental analysis job.
3. The repository scanner inventories affected files, applies ignore and eligibility rules, and submits analyzable content to the AST/parser layer.
4. The AST/parser layer extracts symbols, imports, APIs, comments, dependencies, documentation fragments, and file metadata into normalized repository entities.
5. The backend persists scan records, file state, parsed entities, and change events in PostgreSQL, removing or updating stale records when files change or disappear.
6. The knowledge graph builder creates or updates relationships among files, modules, symbols, dependencies, features, APIs, and business rules.
7. The embedding engine generates or refreshes pgvector embeddings for changed code, documentation, symbols, and knowledge artifacts, then stores them beside relational data.
8. The reasoning/insights engine consumes parsed entities, graph relationships, embeddings, and historical state to produce refreshed observations and recommendations.
9. FastAPI exposes the persisted repository model, semantic search, graph data, insights, and generated artifacts through API endpoints.
10. The WebSocket event stream broadcasts progress and update notifications throughout the pipeline so the Next.js dashboard can refresh repository health, architecture, timeline, insights, recommendations, and AI chat context.

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
