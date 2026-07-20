# Medulla PRD

## Vision
**Medulla is an AI Staff Engineer that continuously observes, understands, reasons about, and improves software projects.**

### Mission
Medulla continuously observes repositories, builds a living understanding of their architecture and intent, reasons about engineering quality, and proactively helps developers improve them.

## Problem
Software knowledge is fragmented across code, chats, docs, pull requests, and people's memories. Git tracks code, but nothing continuously tracks engineering knowledge and intent.

## Product Principles
1. **Observe** – Continuously monitor repository events.
2. **Understand** – Build structured knowledge from code.
3. **Reason** – Detect risks, architecture drift, technical debt, and opportunities.
4. **Act** – Generate documentation, AI context, recommendations, and engineering reports.

## Target Users
- Solo AI-assisted developers
- Indie hackers
- Startup engineering teams
- Open-source maintainers
- Engineering managers

## Core User Journey
Repository → Observe → Understand → Reason → Act

### MVP
- Repository indexing
- Live file watching
- Knowledge graph
- Architecture visualization
- AI project chat
- Living documentation
- AI context export
- Engineering insights dashboard

### MVP Scope and Phasing

#### Phase 1: Repository indexing and CLI scan

**In-scope capabilities**
- Repository indexing that creates an initial project understanding from the current repository state.
- Live file watching that keeps Medulla aware of repository changes after the initial scan.

**Out-of-scope capabilities**
- Knowledge graph generation, architecture visualization, AI chat, living documentation, context export, and dashboard insights.
- Proactive recommendations beyond confirming that repository indexing and updates are working.

**User-facing acceptance criteria**
- A user can point Medulla at a repository and see that the repository has been indexed.
- A user can change a file and see that Medulla recognizes the incremental update.

**Success metric**
- Repository indexed in <60 seconds and incremental updates in <2 seconds.

#### Phase 2: Knowledge extraction and storage

**In-scope capabilities**
- Knowledge graph creation from indexed repository content.
- Architecture visualization based on extracted project knowledge.
- Living documentation generated from the repository understanding.

**Out-of-scope capabilities**
- AI project chat, AI context export, and engineering insights dashboard.
- Proactive engineering insights or recommendations.

**User-facing acceptance criteria**
- A user can view structured project knowledge derived from the repository.
- A user can view an architecture visualization that reflects the indexed project.
- A user can access living documentation generated automatically from repository knowledge.

**Success metric**
- Living documentation generated automatically.

#### Phase 3: AI project chat and context export

**In-scope capabilities**
- AI project chat for asking questions about the repository.
- AI context export that packages project understanding for use in AI-assisted development workflows.

**Out-of-scope capabilities**
- Engineering insights dashboard and proactive recommendations.
- New repository indexing, visualization, or documentation capabilities beyond what earlier phases provide.

**User-facing acceptance criteria**
- A user can ask Medulla repository questions and receive answers grounded in the indexed project knowledge.
- A user can export AI-ready project context from Medulla.

**Success metric**
- AI answers repository questions accurately.

#### Phase 4: Dashboard and insights

**In-scope capabilities**
- Engineering insights dashboard that presents project-level findings.
- Proactive surfacing of engineering insights derived from Medulla's repository understanding.

**Out-of-scope capabilities**
- Direct code editing, automated code changes, or workflow automation beyond surfacing insights.
- Capabilities outside the MVP bullets, such as pull request automation or team process analytics.

**User-facing acceptance criteria**
- A user can open a dashboard and review engineering insights about the repository.
- A user can see proactive insights without manually asking a chat question first.

**Success metric**
- Medulla surfaces engineering insights proactively.

## Success Metrics
- Repository indexed in <60 seconds
- Incremental updates in <2 seconds
- Living documentation generated automatically
- AI answers repository questions accurately
- Medulla surfaces engineering insights proactively

## Positioning

Git → Tracks code

Copilot → Writes code

Cursor → Edits code

**Medulla → Understands the project**

## Tagline
**Every repository deserves an AI Staff Engineer.**
