# ThoughtBox

A personal knowledge management system built as a Chrome extension for capturing, organizing, and retrieving information scattered across multiple platforms.

## The Problem

Knowledge workers use dozens of tools daily — Google Docs, Gmail, Calendar, Proton Mail, Evernote, AI chat tools (Claude, ChatGPT, Gemini, Perplexity) — but no single tool provides unified access across all of them. Information gets lost, duplicated, or forgotten. Questions like "What did I learn about X across all my AI conversations?" or "What science facts haven't I used yet?" become impossible to answer without manually searching each platform.

## The Solution

ThoughtBox captures information from any source and organizes it with a multi-dimensional tagging system, AI-powered suggestions, and unified search. Key capabilities:

- **One-click capture** of web pages, AI conversations, emails, and calendar events
- **Multi-dimensional tags** organized by project, person, content type, context, status, and source
- **AI-assisted organization** with automatic tag suggestions and natural language search
- **Daily briefs** summarizing priorities, calendar events, and relevant research
- **Kindle-style highlights** with per-highlight annotations and best-effort re-anchoring
- **Local-first architecture** — all data stored in IndexedDB, nothing leaves your machine unless you export it

## Tag System

Tags are categorized by prefix to enable powerful multi-dimensional filtering:

| Prefix | Purpose | Examples |
|--------|---------|---------|
| `project:*` | What you're working on | `project:ai-development`, `project:sidewalk-science` |
| `person:*` | Who it relates to | `person:cici`, `person:mindy` |
| `type:*` | Content type | `type:tutorial`, `type:article`, `type:ai-conversation` |
| `context:*` | When/where needed | `context:doctor-appointment`, `context:school` |
| `status:*` | Workflow state | `status:to-read`, `status:completed`, `status:unused` |
| `source:*` | Origin platform | `source:claude`, `source:gmail`, `source:web` |

## Development Phases

| Phase | Focus | Status |
|-------|-------|--------|
| 1 — MVP | Chrome extension, manual tagging, IndexedDB, search, export | In progress |
| 2 — AI | AI tag suggestions, natural language search, duplicate detection | Planned |
| 3 — Integration | Google Calendar/Gmail read-only, daily briefs, cloud sync | Planned |
| 4 — Polish | Proton Mail, Evernote import, AI chat parsing, team features | Planned |
| 5 — Snapshots | Screenshots, full HTML snapshots, offline viewing | Planned |

## Prototype

Interactive UI prototypes are in [`resources/prototype/`](resources/prototype/). Open `index.html` in a browser to explore:

- **Dashboard** — Main library view with sidebar, item list, detail panel, and search
- **Capture Modal** — Quick-capture overlay with auto-filled fields and AI tag suggestions
- **Tag Manager** — Table view of all tags with hierarchy, usage stats, and colors
- **Daily Brief** — AI-generated morning summary with priorities, calendar, and suggestions

## Tech Stack

- **Extension:** Chrome (Manifest V3), with Firefox planned for Phase 2
- **Storage:** IndexedDB via Dexie.js (local-first, no server required)
- **Search:** Fuse.js or FlexSearch for full-text search
- **AI:** Claude API for tag suggestions, summarization, and natural language queries
- **Future:** PostgreSQL with pgvector for cloud sync and semantic search

## Project Structure

```
ThoughtBox/
├── CLAUDE.md                  # Project spec and AI assistant instructions
├── resources/
│   ├── ThoughtBox.pdf         # Requirements document v1.0
│   ├── ThoughtBox Claude.pdf  # Brainstorming and design session transcript
│   └── prototype/             # Interactive HTML prototypes
│       ├── index.html         # Prototype index page
│       └── v1-dashboard.html  # Full dashboard prototype
```

## References

- [Requirements Document](resources/ThoughtBox.pdf) — Full user requirements (v1.0)
- [Design Session](resources/ThoughtBox%20Claude.pdf) — Brainstorming transcript with architecture decisions
