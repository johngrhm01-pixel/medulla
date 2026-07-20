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
- Initial scan <60s
- Incremental update <2s
- Search <300ms
- Insight refresh <5s

## Future
Cloud sync, VS Code extension, GitHub integration, organization knowledge graph, MCP server.

## Dashboard API Requirements

| Dashboard page | Required backend endpoint or WebSocket event | Data source | Refresh behavior | Empty state | Error state |
| --- | --- | --- | --- | --- | --- |
| Repository Health | `GET /api/repositories/{repository_id}/health` and `repository.health.updated` WebSocket event | Latest scan summary, knowledge graph metrics, test and dependency analysis, risk findings, and repository metadata stored in PostgreSQL | Fetch on page load; update immediately when `repository.health.updated` is received; allow manual refresh | Show that no health report is available yet and prompt the user to run an initial scan | Show a retryable error message with the last successful health snapshot if available |
| Architecture | `GET /api/repositories/{repository_id}/architecture` and `architecture.updated` WebSocket event | AST parser output, dependency graph, module metadata, architecture diagrams, and architecture drift findings | Fetch on page load; refresh after scans that change dependency or symbol relationships; update live from WebSocket events | Show an empty architecture canvas with guidance to run or complete repository indexing | Show that architecture data could not be loaded and preserve any previously rendered diagram |
| Features | `GET /api/repositories/{repository_id}/features` and `features.updated` WebSocket event | Feature extraction records, code ownership links, tests, documentation references, and knowledge graph feature nodes | Fetch on page load; refresh after scans or documentation generation complete; support manual refresh | Show that no features have been identified yet and explain that Medulla will populate this after analysis | Show a retryable failure message and keep any cached feature list visible when possible |
| Knowledge | `GET /api/repositories/{repository_id}/knowledge` plus `GET /api/repositories/{repository_id}/knowledge/search` and `knowledge.updated` WebSocket event | Knowledge graph entities and relationships, embeddings, generated documentation, APIs, business rules, and decision records | Fetch summary on page load; search on user query; update graph counts and recent entries on `knowledge.updated` | Show an empty knowledge base state with a call to run indexing or add documentation sources | Show search/index loading failures with retry guidance and distinguish unavailable embeddings from no results |
| Timeline | `GET /api/repositories/{repository_id}/timeline` and `timeline.event.created` WebSocket event | Repository event log, file watcher events, scan jobs, commits, insight events, recommendation state changes, and documentation generation events | Fetch paginated timeline on page load; prepend new events in real time; refresh filters when the user changes date or event type | Show that no repository activity has been recorded yet | Show timeline load failure with retry controls and retain already-loaded pages if pagination fails |
| Insights | `GET /api/repositories/{repository_id}/insights` and `insights.updated` WebSocket event | Reasoning engine outputs, complexity analysis, duplicate logic detection, missing test findings, security and performance observations | Fetch on page load; refresh after reasoning jobs complete; allow users to manually request insight refresh | Show that no insights are available yet and indicate whether analysis is still running or has not started | Show insight generation or load errors with retry and links to job diagnostics when available |
| Recommendations | `GET /api/repositories/{repository_id}/recommendations`, `PATCH /api/recommendations/{recommendation_id}`, and `recommendations.updated` WebSocket event | Recommendation engine results, insight-to-action mappings, user disposition state, priority scoring, and related knowledge graph context | Fetch on page load; update recommendation cards after accept, defer, dismiss, or new reasoning results; subscribe to updates | Show that there are no open recommendations and confirm whether the repository has been analyzed | Show action failure inline per recommendation and list-load failures at page level with retry |
| AI Chat | `POST /api/repositories/{repository_id}/chat` and optional `chat.response.delta` WebSocket or Server-Sent Event stream | OpenAI Responses API, embeddings retrieval, knowledge graph context, repository file snippets, generated documentation, and conversation history | Send request on user message; stream answer tokens when available; refresh citations and retrieved context per message | Show an empty chat prompt with suggested repository questions | Show chat failures inline, preserve the user's draft, and provide retry or regenerate controls |
