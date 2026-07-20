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
| Metric name | Target | Measurement start | Measurement end | Repository size assumption | Environment assumption | Degraded-mode behavior |
| --- | --- | --- | --- | --- | --- | --- |
| Initial scan | <60s | User runs `medulla scan` or triggers the first dashboard scan for a repository after dependencies are installed and credentials are configured. | Repository files are parsed, the knowledge graph is persisted, and scan status is reported as complete; remote embedding generation is excluded from this timing and may continue asynchronously. | Up to 10k tracked source/documentation files, excluding ignored directories such as dependency caches, build artifacts, and vendored assets. | Local developer workstation with 8 CPU cores, 16 GB RAM, SSD storage, warm application dependencies, PostgreSQL + pgvector reachable on localhost, and stable network only for optional asynchronous embedding calls. | If the repository exceeds the size assumption or parsing falls behind, complete metadata/AST indexing first, defer embedding and documentation enrichment, surface partial coverage in status, and continue work in the background. |
| Incremental update | <2s | File watcher, Git observer, IDE event, or CLI event receives a single-file change notification. | Changed file metadata, AST facts, affected graph edges, and dashboard/search invalidations are persisted or queued. | Single-file edits in repositories up to 10k tracked source/documentation files. | Same local workstation profile as initial scan, with watcher service already running and database connection pool warm. | If many changes arrive together, batch them, report queue depth, preserve most recent file state, and temporarily disable non-critical insight recomputation until the backlog clears. |
| Search | <300ms | Backend receives a keyword, graph, or hybrid search request. | First page of ranked results is returned to the caller. | Indexed repository up to 10k tracked source/documentation files with embeddings available for at least 90% of indexed chunks. | Warm FastAPI process, warm database indexes, PostgreSQL + pgvector on localhost, and no live embedding generation in the request path. | If vector indexes or embeddings are unavailable, fall back to keyword/metadata search, return degraded-mode metadata with results, and schedule embedding repair asynchronously. |
| Insight refresh | <5s | User requests insight refresh or an incremental scan marks relevant repository facts as changed. | Refreshed insight summary is available through the API/dashboard and stale insight markers are cleared. | Incremental refresh scoped to changed files and their directly affected dependencies in repositories up to 10k tracked source/documentation files. | Warm backend workers and database, previously completed baseline scan, and configured model/embedding providers; LLM-backed enrichment may use cached context. | If provider latency, rate limits, or repository breadth exceed the target, return deterministic static-analysis insights first, mark LLM-generated recommendations as pending, and retry enrichment in the background. |

## Future
Cloud sync, VS Code extension, GitHub integration, organization knowledge graph, MCP server.
