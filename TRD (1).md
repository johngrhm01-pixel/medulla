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

## Knowledge Graph Model

The first implementation stores the knowledge graph directly in PostgreSQL tables. Nodes and edges are persisted as first-class relational records with JSONB metadata fields for type-specific attributes, and pgvector remains available for semantic retrieval. Derived relational tables may be added later for optimized reporting, but they are not the canonical graph store. An external graph database is out of scope for the initial implementation.

### Initial Node Types

| Node type | Description | Minimum required fields |
| --- | --- | --- |
| Repository | A scanned source repository that owns the graph. | `id`, `type`, `name`, `root_path`, `default_branch`, `created_at`, `updated_at` |
| File | A source, config, test, or documentation file in a repository. | `id`, `type`, `repository_id`, `path`, `language`, `content_hash`, `created_at`, `updated_at` |
| Module | A logical code module, package, namespace, or directory-level unit. | `id`, `type`, `repository_id`, `name`, `path`, `language`, `created_at`, `updated_at` |
| Class | A class, struct, interface, trait, or comparable object-oriented construct. | `id`, `type`, `repository_id`, `name`, `qualified_name`, `file_id`, `created_at`, `updated_at` |
| Function | A function, method, procedure, resolver, handler, or comparable executable symbol. | `id`, `type`, `repository_id`, `name`, `qualified_name`, `file_id`, `signature`, `created_at`, `updated_at` |
| API endpoint | A route, RPC method, GraphQL operation, webhook, or public integration surface. | `id`, `type`, `repository_id`, `name`, `protocol`, `method`, `path`, `created_at`, `updated_at` |
| Dependency | An internal or external package, library, service, runtime, or framework dependency. | `id`, `type`, `repository_id`, `name`, `dependency_kind`, `version_constraint`, `created_at`, `updated_at` |
| Feature | A user-facing or system-facing capability inferred from code, docs, or product metadata. | `id`, `type`, `repository_id`, `name`, `description`, `source`, `created_at`, `updated_at` |
| Business rule | A policy, validation, workflow condition, permission rule, or domain constraint. | `id`, `type`, `repository_id`, `name`, `description`, `source`, `created_at`, `updated_at` |
| Document | A README, design document, ADR, runbook, ticket export, or other prose artifact. | `id`, `type`, `repository_id`, `title`, `path`, `document_kind`, `created_at`, `updated_at` |

### Initial Edge Types

| Edge type | Description | Minimum required fields |
| --- | --- | --- |
| contains | Indicates hierarchy or ownership, such as a repository containing a file or a file containing a class. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `created_at`, `updated_at` |
| imports | Indicates that a file, module, class, or function imports another module, file, or dependency. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `import_name`, `created_at`, `updated_at` |
| calls | Indicates that a function, method, endpoint, or job invokes another function or service operation. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `call_site`, `created_at`, `updated_at` |
| implements | Indicates that a class, function, endpoint, or module fulfills an interface, feature, or business rule. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `evidence`, `created_at`, `updated_at` |
| documents | Indicates that a document, comment, or generated artifact describes another graph node. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `evidence`, `created_at`, `updated_at` |
| depends_on | Indicates a runtime, build-time, deployment, or conceptual dependency between nodes. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `dependency_scope`, `created_at`, `updated_at` |
| exposes | Indicates that a module, service, or file exposes an API endpoint, public symbol, or capability. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `exposure_kind`, `created_at`, `updated_at` |
| related_to | Indicates a general relationship when a more specific edge type does not yet apply. | `id`, `type`, `source_node_id`, `target_node_id`, `repository_id`, `relationship_reason`, `created_at`, `updated_at` |

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
