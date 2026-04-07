# ThoughtBox Design Decisions & Architecture Rationale

> Distilled from initial brainstorming sessions. This document captures the key design decisions,
> architecture rationale, and feature evolution for the ThoughtBox project.

---

## Table of Contents

1. [Product Vision & Market Opportunity](#1-product-vision--market-opportunity)
2. [Why a Browser Extension](#2-why-a-browser-extension)
3. [Storage Decision: IndexedDB](#3-storage-decision-indexeddb)
4. [Multi-Dimensional Tag System](#4-multi-dimensional-tag-system)
5. [Highlights Feature (Kindle-style)](#5-highlights-feature-kindle-style)
6. [Daily Brief Feature](#6-daily-brief-feature)
7. [Capture Workflow Design](#7-capture-workflow-design)
8. [AI Integration Strategy](#8-ai-integration-strategy)
9. [Technical Stack](#9-technical-stack)
10. [Commercial Strategy](#10-commercial-strategy)
11. [Competitive Analysis](#11-competitive-analysis)
12. [Key Architecture Decisions Summary](#12-key-architecture-decisions-summary)
13. [Open Design Questions](#13-open-design-questions)

---

## 1. Product Vision & Market Opportunity

### The Problem

Information is scattered across 10+ tools: Google Docs, Gmail, Google Calendar, Proton Mail, Evernote, various AI chat tools, and more. There is no single place to find, organize, and retrieve knowledge across all of these platforms.

This is not a unique problem. Research from IDC indicates that knowledge workers spend approximately 20% of their time searching for information they need to do their jobs.

### The Insight

ThoughtBox does not aim to replace any of these tools. The key differentiator is **connecting** them. Instead of building yet another notes app or yet another bookmark manager, ThoughtBox acts as a lightweight capture and organization layer that sits on top of existing tools.

### Market Validation

The personal knowledge management (PKM) space has proven demand:

- **Roam Research** -- ~$15M ARR, pioneered bidirectional linking
- **Notion** -- $10B valuation, broad workspace tool
- **Obsidian** -- Profitable with 1M+ users, local-first markdown

These products validate the market but leave gaps: none of them are browser-native, none focus on passive capture from the tools people already use, and most require significant manual effort to organize information.

### Origin

ThoughtBox started from a broader exploration of what software to build. It evolved out of a Tab Manager Chrome extension project. The Tab Manager handled tab organization; ThoughtBox extends that concept into full knowledge management -- capturing not just tabs, but the information within them.

---

## 2. Why a Browser Extension

### Decision: Chrome WebExtension as primary platform

The browser is where most knowledge work happens. A browser extension provides the deepest integration with that workflow.

### Platform Analysis

| Platform | Pros | Cons | Decision |
|----------|------|------|----------|
| **Chrome Extension** | Largest market share, rich APIs, background scripts, local DB access | Chrome-only initially | **Primary target** |
| **Firefox Extension** | WebExtensions API shares ~95% of code with Chrome | Smaller market share | **Phase 2** |
| **Safari Extension** | Covers macOS/iOS users | Requires separate development, $99/year Apple Developer account | **Deferred** |
| **Bookmarklet** | Works on all browsers including Safari | Very limited capabilities -- no background scripts, no local storage, no persistent UI | **Lite version later** |

### Why Not a Bookmarklet First?

A bookmarklet was considered as a universal alternative. While bookmarklets work everywhere (including Safari), they are fundamentally limited:

- No background scripts or persistent state
- No access to browser APIs (tabs, storage, history)
- No ability to inject content scripts for highlights
- No rich popup or sidebar UI
- Cannot run when the browser starts

### Hybrid Approach

Build the full extension first, then add a bookmarklet as a "lite version" for quick capture on unsupported browsers. The bookmarklet would handle basic save-a-link functionality and sync via export/import.

### Extension Capabilities Leveraged

- **Background scripts** -- process data, manage state, listen for events
- **Content scripts** -- inject highlight functionality into any web page
- **Popup/sidebar UI** -- rich interface without leaving the current page
- **IndexedDB access** -- local database with hundreds of GB capacity
- **Browser API integration** -- tabs, tab groups, history, bookmarks

---

## 3. Storage Decision: IndexedDB

### Decision: IndexedDB for MVP, with a clear evolution path

### Alternatives Evaluated

| Option | Capacity | Pros | Cons | Verdict |
|--------|----------|------|------|---------|
| **localStorage** | ~5 MB | Simple key-value API | Far too small, string-only | Rejected |
| **Chrome storage API** | 5-10 MB | Syncs across devices | Too small for knowledge base | Rejected |
| **SQLite/WASM** | Unlimited | Full SQL, mature | Overkill for MVP, complex setup | Deferred |
| **Cloud database** | Unlimited | Multi-device, scalable | Privacy concerns, requires server, needs auth | Deferred |
| **IndexedDB** | Hundreds of GB | Browser-native, stores JS objects, offline, zero setup | Clunky API, no native FTS, per-browser isolation | **Chosen** |

### Why IndexedDB Wins for MVP

1. **Zero infrastructure** -- no server, no account, no setup
2. **Privacy-first** -- data never leaves the user's machine
3. **Sufficient capacity** -- hundreds of GB, far more than needed
4. **Native object storage** -- stores JavaScript objects directly (no serialization)
5. **Offline by default** -- works without internet connection
6. **Browser-native** -- no external dependencies for storage itself

### Addressing IndexedDB Limitations

| Limitation | Mitigation |
|-----------|------------|
| Clunky callback-based API | Use **Dexie.js** as a wrapper (Promise-based, clean syntax) |
| No native full-text search | Add **Fuse.js** (fuzzy search) or **FlexSearch** (fast indexed search) |
| Per-browser data isolation | JSON export/import for portability and backup |
| No cross-device sync | Deferred to cloud phase; JSON export bridges the gap |

### Storage Evolution Path

```
Phase 1: IndexedDB (local, single browser)
    |
Phase 2: IndexedDB + JSON export/import (cross-browser portability)
    |
Phase 3: PostgreSQL + pgvector (cloud sync, AI embeddings, semantic search)
    |
Phase 4: PostgreSQL + Redis cache (performance at scale)
```

---

## 4. Multi-Dimensional Tag System

### Decision: Prefix-based multi-dimensional tags, evolving toward a DAG

### The Core Problem

Information rarely belongs to a single category. A research article might be relevant to a project, associated with a person, needed for a specific context, and have a particular status. Traditional folder hierarchies force a single classification. Tags solve this, but flat tags become unmanageable at scale.

### Tag Design: Prefix-Based Dimensions

Tags use prefixes to indicate their dimension:

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `project:*` | What you are working on | `project:ai-development`, `project:home-renovation` |
| `person:*` | Who it relates to | `person:doctor`, `person:accountant` |
| `type:*` | Content classification | `type:tutorial`, `type:reference`, `type:receipt` |
| `context:*` | When or where it is needed | `context:tax-season`, `context:meeting-prep` |
| `status:*` | Workflow state | `status:to-read`, `status:in-progress`, `status:completed` |
| `source:*` | Auto-generated origin | `source:claude`, `source:gmail`, `source:web` |

### Tag Metadata

Each tag is a first-class entity with its own properties:

```javascript
{
  tagId: "project:ai-development",
  type: "project",
  displayName: "AI Development",
  color: "#3b82f6",
  icon: "...",
  relatedTags: ["project:ai-music", "project:ai-coding"],
  parentTag: "project:ai",   // Phase 2: single parent
  // parentTags: [...]        // Phase 3: multiple parents (DAG)
  itemCount: 47,
  metadata: { status: "active", priority: "high" }
}
```

### Tag Hierarchy Evolution

| Phase | Structure | Description |
|-------|-----------|-------------|
| Phase 1 | Flat tags | No hierarchy, just prefixed names |
| Phase 2 | Single parent | Simple tree -- each tag has at most one parent |
| Phase 3 | DAG | Multiple parents per tag, drop-order determines primary hierarchy, cycle prevention required |

### Smart Collections

Built-in virtual collections that require no manual setup:

- **Untagged** -- items with no tags (needs attention)
- **Recent** -- recently captured or accessed items
- **Popular** -- most frequently accessed items
- **Orphaned** -- tags with no items assigned

### Planned Wildcard Search

- `*` matches any tag name at one level (e.g., `project.*.today`)
- `?` matches a single character (e.g., `project.bug??.backend`)
- `**` matches across multiple levels (Phase 2+)

### Tag Architecture Options

Three architectural approaches for the tag system are being evaluated — see [Tag Architecture Options](tag_architecture_options.md) for the full analysis:

- **Option A (Facets):** Prefixes are hard-coded facets; hierarchy only within facets
- **Option B (Pure DAG):** All tags are equal nodes in a universal DAG; `kind` is just metadata
- **Option C (Hybrid, recommended):** Tags are the universal primitive with an optional `kind` property that drives views but does not constrain the DAG

The decision on which approach to adopt is pending.

### AI-Powered Tag Features (Phase 2+)

- **Auto-suggestion** -- analyze URL, title, and notes to suggest 3-5 tags with confidence levels
- **Learning** -- system improves suggestions based on accept/reject patterns
- **Pattern discovery** -- AI suggests tag merges and new tags when it detects emerging patterns
- **Conditional tags** -- rules-based auto-tagging (e.g., "anything from docs.google.com gets `source:google-docs`")

---

## 5. Highlights Feature (Kindle-style)

### Decision: Kindle-style text highlighting with best-effort anchoring

### Concept

Users can select text on any web page and save the highlighted passage along with a per-highlight annotation. On revisit, highlights are re-rendered on the page using best-effort anchoring.

### Anchoring Strategy

Each highlight stores multiple anchoring signals:

```javascript
{
  text: "The actual highlighted passage",
  note: "Why this matters...",
  anchor: {
    textBefore: "preceding text for context...",
    textAfter: "...following text for context",
    xpath: "/html/body/article/p[3]",
    charOffset: 142
  }
}
```

The system tries anchoring methods in order of reliability. If the exact position is not found, it falls back to fuzzy text matching.

### Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Best-effort anchoring only (v1) | Full robustness against page redesigns is not needed initially. Most pages the user highlights will not change dramatically. |
| Per-highlight notes separate from page-level notes | Different granularity serves different purposes. A page note summarizes; a highlight note explains why a specific passage matters. |
| Read/unread independent of highlights | Just because you highlighted something does not mean you have "read" the page. Status is a manual toggle. |
| Highlights nested within captured items | A highlight always belongs to a specific URL/page. No orphan highlights. |
| Tagging highlights directly deferred to P2+ | Start simple. If usage patterns show highlights need their own tags, add it later. |

---

## 6. Daily Brief Feature

### Decision: Cross-platform daily summary, inspired by Google's Copilot for Workspace

### Motivation

Google's Copilot for Workspace (CC) provides daily briefings, but only for Google products and only for enterprise users. ThoughtBox's daily brief works across all connected tools -- including privacy-focused services like Proton Mail and various AI chat platforms.

### Brief Structure

| Section | Content |
|---------|---------|
| **Top Priority** | Items tagged `status:urgent` or `context:today` |
| **Today's Calendar** | Upcoming events with linked research and prep materials |
| **Important Emails** | Flagged or high-priority email links |
| **Relevant Research** | Recently captured items related to today's tasks |
| **Suggestions** | AI-generated recommendations (e.g., "You saved 3 articles about X but haven't read them") |

### Implementation Phases

1. **Manual** -- aggregates tagged items + calendar data, simple template
2. **AI-enhanced** -- Claude analyzes items and generates natural-language summary
3. **Full integration** -- pulls from Gmail, Calendar, Proton Mail APIs with smart prioritization

### Delivery Options

- Email at a configurable time (e.g., 7 AM daily)
- Web dashboard within the extension
- Mobile notification (future, requires companion app)

---

## 7. Capture Workflow Design

### Decision: Metadata-only capture with source linking (no content duplication)

### Evolution

The capture concept evolved from a "Delicious++" bookmarking model to a richer knowledge capture system. The key insight: ThoughtBox does not need to store the content itself. It stores metadata, tags, notes, and highlights -- then links back to the original source.

### Supported Capture Types

| Source | What is Captured | How |
|--------|-----------------|-----|
| Web pages | URL, title, auto-detected metadata | Extension button or keyboard shortcut |
| Google Docs/Sheets | Document URL, title, doc ID | Extension detects Google Docs URLs |
| AI conversations | Copy/paste of key exchanges | Manual paste into notes field |
| Email | Link to email thread | Extension or manual URL save |
| Calendar events | Event link, date, attendees | Integration or manual save |
| Local files | File path, name, metadata | Manual entry (future: drag and drop) |

### Key Workflow Decisions

| Decision | Rationale |
|----------|-----------|
| Save metadata, not content | Avoids OAuth complexity, respects source ownership, sidesteps storage bloat |
| Clicking saved items opens original source | No need to re-render or cache content; the source of truth stays in the original tool |
| Extension auto-detects content type | Reduces manual input; knows if you are on Gmail vs Google Docs vs a blog |
| "Save & Close Tab" power feature | Common workflow: save the link, close the tab, come back later |
| No API integration needed for MVP | Just saving URLs and metadata works without any OAuth or API keys |

---

## 8. AI Integration Strategy

### Decision: AI as an optional enhancement layer, not a core dependency

### Principles

1. **ThoughtBox must work fully without AI** -- AI features are additive
2. **User can disable AI entirely** -- privacy control
3. **Local processing where possible** -- minimize data sent to external APIs
4. **Learn from user behavior** -- improve suggestions over time

### AI Capabilities (Planned)

| Capability | AI Provider | Phase |
|-----------|-------------|-------|
| Tag suggestion (analyze URL + title + notes) | Claude API | Phase 2 |
| Natural language search | OpenAI embeddings | Phase 2 |
| Content summarization | Claude API | Phase 2 |
| Duplicate detection | Embeddings similarity | Phase 2 |
| Tag merge suggestions | Claude API | Phase 3 |
| Daily brief generation | Claude API | Phase 3 |
| Semantic search across all captures | pgvector | Phase 3 |

### Tag Suggestion Flow

```
User saves a page
    --> Extension sends URL + title + notes + existing tags to Claude API
    --> Claude returns 3-5 suggested tags with confidence levels
    --> User accepts, rejects, or modifies suggestions
    --> System records the decision for future learning
```

---

## 9. Technical Stack

### Recommended Stack by Phase

| Layer | MVP (Phase 1) | Growth (Phase 2-3) | Scale (Phase 4+) |
|-------|--------------|-------------------|------------------|
| **Frontend** | Chrome Extension (HTML/CSS/JS) | React or Vue components | React + design system |
| **Storage** | IndexedDB via Dexie.js | IndexedDB + SQLite export | PostgreSQL + pgvector |
| **Search** | Fuse.js or FlexSearch | FlexSearch + AI embeddings | PostgreSQL FTS + pgvector |
| **Backend** | None (local only) | Node.js or Java Spring Boot | Microservices |
| **AI** | None | Claude API + OpenAI embeddings | Multi-model orchestration |
| **Auth** | None | OAuth 2.0 (Google) | OAuth 2.0 + SSO |
| **Export** | JSON | JSON + CSV | JSON + CSV + API |

### Library Choices

| Library | Purpose | Why This One |
|---------|---------|-------------|
| **Dexie.js** | IndexedDB wrapper | Promise-based, clean API, active maintenance, good docs |
| **Fuse.js** | Fuzzy text search | Lightweight, no index build step, good for small-medium datasets |
| **FlexSearch** | Full-text search | Faster than Fuse.js for larger datasets, supports indexing |

---

## 10. Commercial Strategy

### Pricing Tiers

| Tier | Price | Features |
|------|-------|----------|
| **Free** | $0 | 100 captures/month, basic search, local storage only, manual tagging |
| **Pro** | $10/month | Unlimited captures, AI tag suggestions, email integration, cloud sync |
| **Power** | $25/month | Multiple AI tool integrations, team sharing, analytics dashboard |
| **Enterprise** | Custom | Custom integrations, SSO, admin controls, priority support |

### Go-to-Market

- Launch as a free Chrome extension to build user base
- Prove value with local-only features before asking for payment
- AI features and cloud sync are the natural upgrade triggers
- Privacy-first positioning differentiates from competitors that require cloud accounts

---

## 11. Competitive Analysis

| Competitor | Strengths | Weaknesses | ThoughtBox Opportunity |
|-----------|-----------|------------|----------------------|
| **Capacities** | AI-native, clean UI | No browser capture, no multi-AI-tool support | Capture workflow + AI tool integration |
| **Heptabase** | Visual knowledge graphs | Steep learning curve, no quick capture | Simpler entry point |
| **Tana** | Powerful structure | Complex, overkill for most users | Simplicity as a feature |
| **Raindrop.io** | Great bookmarking | Weak AI, no knowledge synthesis | AI layer on top of bookmarking |
| **Obsidian** | Local-first, rich plugin ecosystem | No web capture, markdown-centric | Browser-native experience |
| **Mem.ai** | AI-powered notes | Closed ecosystem, no multi-source capture | Open architecture, privacy-first |
| **Notion / Coda** | Rich workspace | Manual organization burden | Passive, intelligent capture |
| **Evernote** | Good web clipper | Weak AI, siloed data | AI organization layer |

### ThoughtBox's Positioning

ThoughtBox sits at the intersection of **bookmarking** (like Raindrop.io), **knowledge management** (like Obsidian), and **AI organization** (like Mem.ai) -- but differentiated by being:

- **Browser-native** -- lives where you already work
- **Privacy-first** -- local storage by default
- **Multi-source** -- connects tools rather than replacing them
- **Low-friction** -- capture is fast, organization is AI-assisted

---

## 12. Key Architecture Decisions Summary

| # | Decision | Choice | Rationale |
|---|----------|--------|-----------|
| 1 | Primary platform | Chrome WebExtension | Rich APIs, largest market share, ~95% code sharing with Firefox |
| 2 | MVP storage | IndexedDB | Zero setup, offline, sufficient for single user, privacy-first |
| 3 | Tag system | Prefix-based multi-dimensional | Solves the "information belongs to multiple contexts" problem |
| 4 | Highlight anchoring | Best-effort (v1) | Full robustness against page redesigns not needed initially |
| 5 | AI provider | Claude API | Best for text analysis and tag suggestion tasks |
| 6 | Full-text search | Fuse.js or FlexSearch | Compensates for IndexedDB's lack of native FTS |
| 7 | Data portability | JSON export/import | Cross-browser portability, human-readable backup |
| 8 | API integrations | Deferred past MVP | Sidesteps OAuth complexity, works offline |
| 9 | Content storage | Metadata + links only | Avoids content duplication, respects source ownership |
| 10 | AI dependency | Optional enhancement | Core product works without AI; users can disable AI entirely |

---

## 13. Open Design Questions

These questions remain open and will be resolved as usage patterns emerge:

1. **Should highlights be taggable entities?**
   Leaning toward P2+ based on actual usage. If users want to tag specific passages differently from the page they came from, the feature will be added.

2. **Tag hierarchy limits**
   - Maximum number of parents per tag (3-5?)
   - Maximum depth of hierarchy (5 levels?)
   - Cycle prevention algorithm needed for DAG structure
   - See [Tag Architecture Options](tag_architecture_options.md) for deeper exploration of architectural approaches

3. **Primary UI surface**
   - Popup (current, like Tab Manager)
   - Side panel (Chrome 114+, keeps page visible)
   - Full page (for library/search/management)
   - Likely all three for different use cases

4. **Permission model**
   - `<all_urls>` needed for highlights on any page
   - `activeTab` is less scary permission-wise but adds friction
   - Trade-off between capability and user trust

5. **Dynamic/JS-rendered content**
   - How to handle highlights on SPAs and dynamically loaded content?
   - MutationObserver? Delay-and-retry? Content hash verification?

6. **Performance at scale**
   - Pagination strategy for large collections
   - Background indexing for search
   - Lazy loading for tag counts and metadata

---

## Development Phases

| Phase | Focus | Key Deliverables |
|-------|-------|-----------------|
| **1 (MVP)** | Basic capture | Chrome extension, manual tagging, IndexedDB storage, list view, search, JSON export |
| **2** | AI enhancement | AI tag suggestions, natural language search, duplicate detection, Firefox port |
| **3** | Integration | Google Calendar/Gmail read-only integration, daily brief, cloud sync |
| **4** | Polish & scale | Proton Mail integration, Evernote import, AI chat parsing, team features |
| **5** | Snapshots | Screenshots, full HTML snapshots, offline viewing |

---

*This document is a living reference. Update it as decisions are made or revisited.*
