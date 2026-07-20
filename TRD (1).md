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

## Performance Targets
- Initial scan <60s
- Incremental update <2s
- Search <300ms
- Insight refresh <5s

## Future
Cloud sync, VS Code extension, GitHub integration, organization knowledge graph, MCP server.
