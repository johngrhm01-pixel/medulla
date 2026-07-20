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

The `medulla` CLI is the primary local developer interface for connecting a repository to Medulla, running scans, querying indexed knowledge, exporting AI-ready context, and troubleshooting local configuration. Commands should emit human-readable text by default and support machine-readable JSON where noted.

### `medulla init`

**Purpose**
- Initialize Medulla configuration for the current repository.
- Create a local project identity and default scan settings.
- Optionally register the repository with the backend service.

**Arguments**
- Required: none.
- Optional:
  - `[path]`: repository path to initialize. Defaults to the current working directory.
  - `--name <name>`: display name for the repository. Defaults to the directory name.
  - `--backend-url <url>`: backend API URL to write into local configuration.
  - `--token <token>`: API token used to register or authenticate the repository.
  - `--local-only`: create local configuration without contacting the backend.
  - `--force`: overwrite an existing Medulla configuration file.

**Example usage**
```bash
medulla init
medulla init ~/src/api --name payments-api --backend-url http://localhost:8000
medulla init --local-only --force
```

**Expected terminal output**
```text
Medulla initialized for payments-api
Config: .medulla/config.toml
Repository ID: repo_01HXYZ...
Backend: http://localhost:8000
Next: run `medulla scan`
```

For `--local-only`:
```text
Medulla initialized in local-only mode
Config: .medulla/config.toml
Next: run `medulla scan --local`
```

**Exit codes**
- `0`: initialization completed successfully.
- `1`: invalid arguments or unsupported repository path.
- `2`: configuration already exists and `--force` was not provided.
- `3`: backend registration failed.
- `4`: unable to write configuration files.

**Backend requirement**
- Can run locally with `--local-only`.
- Requires the backend service when registering the repository or validating `--backend-url` and `--token`.

**Files and configuration read/written**
- Reads `.git/config` to infer repository metadata when available.
- Writes `.medulla/config.toml` with repository ID, name, backend URL, and scan defaults.
- Writes `.medulla/state.json` with initialization metadata.
- May read environment variables such as `MEDULLA_BACKEND_URL` and `MEDULLA_TOKEN`.

### `medulla scan`

**Purpose**
- Scan repository files and convert source, documentation, and dependency metadata into Medulla knowledge records.
- Populate or refresh the knowledge graph, embeddings, and documentation summaries.

**Arguments**
- Required: none.
- Optional:
  - `[path]`: repository path to scan. Defaults to the current working directory.
  - `--full`: ignore incremental state and rescan all supported files.
  - `--since <ref>`: scan files changed since a Git ref, commit SHA, or branch.
  - `--include <glob>`: include only paths matching the glob. Can be repeated.
  - `--exclude <glob>`: exclude paths matching the glob. Can be repeated.
  - `--local`: perform parsing and local cache updates without sending data to the backend.
  - `--json`: emit a JSON summary.
  - `--watch`: keep running and process file changes incrementally.

**Example usage**
```bash
medulla scan
medulla scan --full --exclude "node_modules/**"
medulla scan --since main --json
medulla scan --watch
```

**Expected terminal output**
```text
Scanning repository: payments-api
Mode: incremental
Files scanned: 184
Symbols indexed: 2,341
Embeddings queued: 184
Insights refreshed: 12
Scan completed in 18.4s
```

With `--json`:
```json
{
  "repository": "payments-api",
  "mode": "incremental",
  "filesScanned": 184,
  "symbolsIndexed": 2341,
  "embeddingsQueued": 184,
  "insightsRefreshed": 12,
  "durationMs": 18400
}
```

**Exit codes**
- `0`: scan completed successfully.
- `1`: invalid arguments or repository path.
- `2`: Medulla has not been initialized for the repository.
- `3`: backend request, upload, or indexing failure.
- `4`: local parser, filesystem, or cache failure.
- `5`: scan completed with recoverable warnings.

**Backend requirement**
- Requires the backend service for canonical indexing, embedding generation, insight refresh, and dashboard updates.
- Can run locally with `--local`, but local mode only updates `.medulla` cache files and does not refresh backend-derived insights.

**Files and configuration read/written**
- Reads `.medulla/config.toml`, `.medulla/state.json`, `.gitignore`, and repository files selected by include/exclude rules.
- Reads package manifests and dependency files such as `package.json`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, and lockfiles when present.
- Writes `.medulla/state.json` with last scan metadata.
- Writes `.medulla/cache/` with parser output and incremental scan checkpoints in local mode.

### `medulla status`

**Purpose**
- Show repository connection, scan freshness, indexing progress, and local watcher state.
- Provide a quick health snapshot for the CLI, backend, and dashboard-facing data.

**Arguments**
- Required: none.
- Optional:
  - `[path]`: repository path to inspect. Defaults to the current working directory.
  - `--json`: emit machine-readable status.
  - `--verbose`: include backend latency, queue depth, and recent warning details.
  - `--offline`: report only local state without contacting the backend.

**Example usage**
```bash
medulla status
medulla status --verbose
medulla status --offline --json
```

**Expected terminal output**
```text
Repository: payments-api
Initialized: yes
Backend: connected (42ms)
Last scan: 2026-07-20 13:04:22 UTC
Indexed files: 184
Pending embeddings: 0
Watcher: stopped
Health: ok
```

**Exit codes**
- `0`: status retrieved and repository is healthy.
- `1`: invalid arguments or repository path.
- `2`: Medulla has not been initialized for the repository.
- `3`: backend is unreachable or returned an error.
- `5`: status retrieved but warnings are present, such as stale scans or queued failures.

**Backend requirement**
- Requires the backend service for live connection, indexing queue, dashboard, and insight status.
- Can run locally with `--offline`, limited to `.medulla` state and cache metadata.

**Files and configuration read/written**
- Reads `.medulla/config.toml` and `.medulla/state.json`.
- Reads `.medulla/cache/` metadata when `--offline` or `--verbose` is used.
- Does not write repository files; may update transient CLI telemetry if enabled in user-level configuration.

### `medulla ask`

**Purpose**
- Ask natural-language questions about the repository using indexed code knowledge, documentation, architecture summaries, and embeddings.
- Return answers with source references where possible.

**Arguments**
- Required:
  - `<question>`: the question to answer. The value can be a quoted string or read from standard input when `-` is provided.
- Optional:
  - `--path <path>`: repository path. Defaults to the current working directory.
  - `--scope <glob>`: restrict retrieval to matching files or directories. Can be repeated.
  - `--format <text|markdown|json>`: output format. Defaults to `markdown`.
  - `--top-k <number>`: maximum number of retrieved context chunks. Defaults to backend configuration.
  - `--no-stream`: disable streaming responses.
  - `--model <name>`: model override when permitted by backend policy.

**Example usage**
```bash
medulla ask "Where is authentication implemented?"
medulla ask "Summarize payment retry behavior" --scope "src/payments/**"
printf 'What changed in the API layer?\n' | medulla ask - --format json
```

**Expected terminal output**
```markdown
Authentication is implemented in the API middleware and session service.

Sources:
- src/api/middleware/auth.ts
- src/services/session.ts

Confidence: high
```

**Exit codes**
- `0`: answer generated successfully.
- `1`: invalid arguments, missing question, or unsupported output format.
- `2`: Medulla has not been initialized for the repository.
- `3`: backend retrieval or model request failed.
- `4`: unable to read standard input or local configuration.
- `5`: answer generated with warnings, such as stale index data or partial retrieval.

**Backend requirement**
- Requires the backend service because repository retrieval, embeddings, and model calls are backend-owned.
- Does not support fully local answering unless a future local model provider is configured.

**Files and configuration read/written**
- Reads `.medulla/config.toml` for repository identity, backend URL, and authentication settings.
- Reads `.medulla/state.json` to warn about stale scans.
- Does not write project files.
- May write shell history outside the repository depending on the user's shell; sensitive questions should be passed through standard input when appropriate.

### `medulla export`

**Purpose**
- Export repository context for AI tools, pull request reviews, documentation generation, or offline analysis.
- Produce a portable bundle containing selected summaries, source references, dependency metadata, and optional raw snippets.

**Arguments**
- Required: none.
- Optional:
  - `[path]`: repository path to export. Defaults to the current working directory.
  - `--format <markdown|json|zip>`: export format. Defaults to `markdown`.
  - `--output <file>`: destination file. Defaults to standard output for text formats and `medulla-export.zip` for zip format.
  - `--scope <glob>`: limit export contents to matching paths. Can be repeated.
  - `--include-raw`: include raw source snippets where policy allows.
  - `--max-tokens <number>`: approximate token budget for text exports.
  - `--refresh`: request a backend refresh before exporting.
  - `--local`: build the export from local cache without contacting the backend.

**Example usage**
```bash
medulla export --format markdown --output context.md
medulla export --format json --scope "src/api/**" --max-tokens 12000
medulla export --format zip --output medulla-context.zip --include-raw
```

**Expected terminal output**
```text
Export created: context.md
Format: markdown
Files summarized: 184
Approx tokens: 11,820
Sources included: yes
```

When writing to standard output, the command prints the export body instead of the summary.

**Exit codes**
- `0`: export completed successfully.
- `1`: invalid arguments, unsupported format, or invalid output path.
- `2`: Medulla has not been initialized for the repository.
- `3`: backend export or refresh failed.
- `4`: unable to read local cache or write the output file.
- `5`: export completed with warnings, such as truncation due to `--max-tokens`.

**Backend requirement**
- Requires the backend service for the freshest generated summaries, embeddings-aware context selection, and `--refresh`.
- Can run locally with `--local` if a scan cache exists, but output may be stale or less complete.

**Files and configuration read/written**
- Reads `.medulla/config.toml`, `.medulla/state.json`, and `.medulla/cache/`.
- Reads repository files when `--include-raw` or local cache misses require source snippets.
- Writes the file specified by `--output`, or `medulla-export.zip` by default for zip exports.
- Does not modify Medulla configuration unless `--refresh` updates scan metadata through the backend.

### `medulla doctor`

**Purpose**
- Diagnose CLI installation, local repository configuration, backend connectivity, database reachability, embedding configuration, and common scanner issues.
- Provide actionable remediation steps for failed checks.

**Arguments**
- Required: none.
- Optional:
  - `[path]`: repository path to diagnose. Defaults to the current working directory.
  - `--json`: emit machine-readable diagnostics.
  - `--fix`: apply safe automatic repairs, such as creating missing cache directories.
  - `--backend-url <url>`: override configured backend URL for connectivity checks.
  - `--verbose`: include detailed check timings and error payloads.

**Example usage**
```bash
medulla doctor
medulla doctor --verbose
medulla doctor --fix --json
```

**Expected terminal output**
```text
Medulla doctor
✓ CLI version: 0.1.0
✓ Repository initialized: .medulla/config.toml
✓ Git repository detected
✓ Backend reachable: http://localhost:8000
✓ Database status: ok
✓ Embeddings provider: configured
! Last scan is stale: 3 days old

Result: warnings found
Suggested next step: run `medulla scan`
```

**Exit codes**
- `0`: all checks passed.
- `1`: invalid arguments or unsupported diagnostic target.
- `2`: repository is not initialized or required local configuration is missing.
- `3`: backend, database, or model provider check failed.
- `4`: local filesystem repair failed or required files are unreadable.
- `5`: diagnostics completed with warnings.

**Backend requirement**
- Can run local checks without the backend service.
- Requires the backend service for API, database, queue, dashboard, and model-provider checks.
- With no backend available, exits `5` when local checks pass but remote checks are skipped by configuration, or `3` when backend checks are required and fail.

**Files and configuration read/written**
- Reads `.medulla/config.toml`, `.medulla/state.json`, `.medulla/cache/`, `.git/config`, and supported dependency manifests.
- Reads environment variables such as `MEDULLA_BACKEND_URL`, `MEDULLA_TOKEN`, `OPENAI_API_KEY`, and database-related settings exposed to the backend health endpoint.
- Writes missing `.medulla/cache/` directories or repaired state fields only when `--fix` is provided.
- Does not modify source files.

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
