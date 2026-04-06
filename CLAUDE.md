# ThoughtBox - CLAUDE.md

## What is ThoughtBox?

A Chrome extension for personal knowledge management. It started as an evolution of a Tab Manager extension and grew into a standalone project for capturing, organizing, and retrieving information scattered across multiple platforms (Google Docs, Gmail, Calendar, Proton Mail, Evernote, AI chat tools).

Built for personal use first with potential for commercialization later.

## Core Concepts

### Captured Items (Bookmarks/Links)
- Save web pages with URL, title, notes, tags, and metadata
- Track read/unread status (manual toggle, independent of highlights)
- Support page-level notes

### Kindle-style Highlights
- Select text on web pages and save with per-highlight annotations
- Re-highlight text on revisit using best-effort anchoring (not required to survive page redesigns in v1)
- Anchoring uses: textBefore, textAfter, xpath, charOffset
- Highlights are nested within a captured item (link)
- Per-highlight notes are separate from page-level notes
- Search within highlights is planned for P2

### Tag System (Multi-dimensional, evolving toward DAG)
Tags are categorized by prefix:
- `project:*` - what you're working on (e.g., `project:ai-development`)
- `person:*` - who it relates to (e.g., `person:family-member`)
- `type:*` - content type (e.g., `type:tutorial`, `type:reference`)
- `context:*` - when/where needed (e.g., `context:doctor-appointment`)
- `status:*` - workflow state (e.g., `status:to-read`, `status:completed`)
- `source:*` - auto-generated origin (e.g., `source:claude`, `source:gmail`)

**Tag evolution plan:**
- Phase 1: Flat tags only (no hierarchy)
- Phase 2: Single parent per tag (simple tree)
- Phase 3: Tag DAG (multiple parents, drop-order determines primary hierarchy)

**Wildcard search planned:**
- `*` = match any tag name at one level (e.g., `tab-manager.*.today`)
- `?` = single character wildcard (e.g., `tab-manager.bug??.backend`)
- `**` = match across multiple levels (Phase 2+)

### Taggable Entities
- Links/Pages: P1 (core)
- Highlights: P2+ (open design question)
- Tags themselves can have parent tags (DAG structure)

## Technical Architecture

### Storage
- **MVP:** IndexedDB (browser-native, handles complex nested data, hundreds of GB capacity)
- **Libraries:** Dexie.js or idb (IndexedDB wrapper), Fuse.js or FlexSearch (full-text search)
- **Future:** PostgreSQL with cloud sync and vector embeddings

### Permissions
```json
{
  "permissions": ["tabs", "tabGroups", "storage", "activeTab"],
  "host_permissions": ["<all_urls>"],
  "content_scripts": [{"matches": ["<all_urls>"], "js": ["content.js"]}]
}
```
Note: `<all_urls>` needed for highlights. Consider `activeTab` alternative for less scary permission warning (more friction but better privacy perception).

### Data Models

**Captured Item:**
```javascript
{
  id: "uuid",
  url: "https://example.com/article",
  title: "Article Title",
  notes: "User's notes...",
  tags: ["project:ai-development", "type:tutorial"],
  status: "unread", // independent of highlight activity
  highlights: [
    {
      id: "h1",
      text: "The actual highlighted passage",
      note: "Per-highlight annotation",
      anchor: {
        textBefore: "surrounding context...",
        textAfter: "...more context",
        xpath: "/html/body/article/p[3]",
        charOffset: 142
      },
      created: "2025-01-29T10:30:00Z"
    }
  ],
  capturedAt: "2025-01-18T10:30:00Z",
  lastAccessedAt: "2025-01-18T14:22:00Z"
}
```

**Tag:**
```javascript
{
  tagId: "project:ai-development",
  type: "project",
  displayName: "AI Development",
  color: "#3b82f6",
  icon: "🤖",
  relatedTags: ["project:ai-music", "project:ai-coding"],
  parentTag: "project:ai", // single parent in Phase 2; parents[] array in Phase 3
  itemCount: 47,
  metadata: { status: "active", priority: "high" }
}
```

**Snapshot (Phase 5):**
```javascript
{
  id: "auto-increment",
  bookmarkId: "indexed",
  screenshot: Blob,
  html: "string", // full page HTML for offline viewing
  createdAt: "timestamp"
}
```

### UI Strategy
- Start with popup (like Tab Manager)
- Evaluate side panel (Chrome 114+) for Phase 2+
- Separate full-page management interface for library/search

## Development Phases

| Phase | Focus | Key Features |
|-------|-------|-------------|
| 1 (MVP) | Basic capture | Chrome ext, manual tagging, IndexedDB, list view, search, export |
| 2 | AI Enhancement | AI tag suggestions, NL search, duplicate detection, Firefox |
| 3 | Integration | Google Calendar/Gmail (read-only), daily brief, cloud sync |
| 4 | Polish & Scale | Proton Mail, Evernote import, AI chat parsing, team features |
| 5 | Snapshots | Screenshots, full HTML snapshots, offline viewing |

## Key Decisions Made

1. **IndexedDB over alternatives** - browser-native, no server needed, handles nested data, hundreds of GB
2. **Best-effort highlight anchoring** - no need to survive page redesigns for v1
3. **Per-highlight annotations** - separate from page-level notes
4. **Read/unread independent of highlights** - manual toggle only
5. **Tags on links first** - extending to highlights is P2+
6. **Local-first architecture** - privacy-first, no data leaves machine unless exported
7. **Fork from Tab Manager** - reuse core architecture (manifest, popup structure), add IndexedDB layer
8. **ThoughtBox as separate project** - Tab Manager stays lightweight standalone

## Migration from Tab Manager

- Tab Manager remains standalone (tab management, recently closed, duplicates, group filtering)
- ThoughtBox forks core architecture and adds IndexedDB + bookmark management
- Potential integration: "Save this tab to ThoughtBox" button in Tab Manager
- Shared UI patterns and styling

## Open Design Questions

1. Should highlights be taggable entities? (leaning toward P2+ based on usage patterns)
2. Tag hierarchy limits: max parents (3-5?), max depth (5 levels?), cycle prevention?
3. UI: popup vs side panel vs full page for primary interface?
4. Permission model: `<all_urls>` vs `activeTab` trade-off?
5. How to handle dynamic/JS-rendered content for highlights?
6. Performance strategy for large datasets (pagination, background indexing)?

## Project References

- **Requirements Doc:** Google Docs (ThoughtBox User Requirements Document v1.0)
- **Feature Ideas:** `docs/FEATURE_IDEAS.md` in the repo
- **Claude Project:** This project contains the full brainstorming and requirements history
- **Related repo:** Tab Manager Chrome extension (the predecessor/sibling project)

## Coding Preferences

- The developer typically writes Java but this project is JavaScript/TypeScript (Chrome extension)
- Prefer clear, readable code over clever abstractions
- Start simple, evolve when complexity is justified
- Document decisions and rationale
