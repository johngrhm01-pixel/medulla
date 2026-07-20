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


## Privacy and Trust Requirements
- Users must have explicit control over what repositories, directories, and file types are indexed, with clear controls to pause, rescan, or remove indexed content.
- Private source code must be handled transparently: the product should explain what code is read, where derived data is stored, and when any content may be sent to AI services.
- Secrets, credentials, generated artifacts, dependency folders, and files excluded by `.gitignore` or user-defined ignore rules must be excluded from indexing, embeddings, and AI context export.
- For the MVP, Medulla should default to local-first behavior where applicable: repository metadata, embeddings, generated docs, and analysis outputs should remain on the user's machine unless the user opts into an external integration or API-backed feature.

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
