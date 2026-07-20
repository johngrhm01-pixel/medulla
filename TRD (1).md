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


## Security and Privacy Design
- Respect `.gitignore` by default and support a Medulla-specific ignore file or configuration for additional exclusions, including glob patterns for paths, file types, generated assets, and vendor directories.
- Run secret detection before embedding generation, documentation generation, AI chat context assembly, or any external AI API call; redact or skip suspected secrets and surface actionable warnings to the user.
- Define a data retention policy for embeddings and generated documentation: store them locally for the MVP, tie records to repository identity and revision metadata, and provide commands or UI controls to delete, rebuild, or expire derived data.
- Handle API keys through environment variables or OS-level secret storage; never persist plaintext keys in repository files, generated docs, logs, exported AI context, or the local database.
- Protect the local database with least-privilege file permissions, clear storage location disclosure, and safe deletion behavior when a repository is removed or a user resets Medulla state.
- Keep audit and application logs bounded: record operational events, indexing decisions, warnings, and errors without logging source snippets, secrets, embeddings, prompts, responses, or private repository content by default.

## Performance Targets
- Initial scan <60s
- Incremental update <2s
- Search <300ms
- Insight refresh <5s

## Future
Cloud sync, VS Code extension, GitHub integration, organization knowledge graph, MCP server.
