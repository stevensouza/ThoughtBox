# ThoughtBox - Personal Knowledge Management System

## User Requirements Document v1.0

---

## 1. Executive Summary

### 1.1 Problem Statement

Knowledge workers struggle to organize and retrieve information scattered across multiple platforms: Google Docs, Gmail, Google Calendar, Proton Mail, Proton Docs, Evernote, browser tabs, and various AI tools (Claude, ChatGPT, Gemini, Perplexity, Copilot, Lumina). Current tools either require manual organization (Notion, Evernote) or are platform-specific (Google search, email search). No single tool provides unified access, intelligent organization, and AI-powered retrieval across all these sources.

### 1.2 Solution Overview

A browser-based knowledge management system that:

- Captures information from any source (web pages, documents, emails, AI conversations)
- Uses AI to automatically organize and tag content
- Provides unified search and retrieval across all captured information
- Generates daily briefs showing what is important for the day
- Works across multiple browsers (Chrome, Firefox, initially)
- Organizes by multi-dimensional tags (projects, people, contexts, types)

### 1.3 Target User

Initially built for personal use, with potential for commercialization. The primary user is a knowledge worker who:

- Manages multiple projects simultaneously
- Uses diverse information sources and tools
- Needs quick access to previously captured information
- Benefits from AI-assisted organization and retrieval
- Values privacy and data control

---

## 2. Current Workflow and Pain Points

### 2.1 Daily Activities

- Managing tasks via Google Docs todo list
- Tracking projects via spreadsheets
- Email management (Gmail + Proton Mail)
- Calendar coordination (Google Calendar)
- Research via web browsers
- AI tool usage (Claude, ChatGPT, Gemini, Perplexity, Copilot, Lumina)
- Note-taking (Evernote, Proton Docs, Google Docs)

### 2.2 Current Pain Points

- Information scattered across 10+ different tools
- No unified search across all sources
- Difficult to track what research has been done on a topic
- Hard to connect related information across different tools
- Todo lists do not connect to supporting research and emails
- Cannot easily answer questions like:
  - "What did I learn about X across all my AI conversations?"
  - "What resources on topic Y have I not yet used?"
  - "What is important for today across all my systems?"

### 2.3 Example Use Cases

**Research Tracking Project:**
- Research facts and references via web and AI tools
- Track which resources have been used versus unused
- Maintain a running collection of gathered material
- Need to avoid duplicating previously used content

**Family Activity Planning:**
- Information spans calendar events, emails, web research, and notes
- Includes tasks like registering a child for activities, planning events
- Related information scattered across multiple platforms

**AI Development Work:**
- Research across multiple AI platforms
- Code snippets, documentation, tutorials
- Need to connect learnings from different sources

---

## 3. Core Features

### 3.1 Information Capture

#### 3.1.1 Capture Methods

- **Browser Extension (primary method)**
  - One-click capture of current page
  - URL, title, screenshot, and timestamp automatically saved
  - User adds notes and tags via popup interface
  - Right-click menu for quick capture
  - Keyboard shortcuts for power users

#### 3.1.2 Supported Content Types

| # | Content Type | Details |
|---|-------------|---------|
| 1 | Web Pages | URL, title, full text extraction, screenshot, author, publish date, reading time |
| 2 | Google Workspace | Direct links to Docs, Sheets, Slides with metadata |
| 3 | AI Conversations | Copy/paste from Claude, ChatGPT, Gemini, etc. with auto-tagging |
| 4 | Email | Gmail and Proton Mail links with subject, sender, date metadata |
| 5 | Calendar Events | Link to event with date, time, attendees, location |
| 6 | Local Files (future) | File path and metadata |

#### 3.1.3 Capture Workflow

```
User sees interesting content
    |
    v
Clicks extension icon or uses keyboard shortcut
    |
    v
Popup appears with:
  - Auto-filled title, URL, source
  - Text area for notes
  - AI-suggested tags (accept/reject)
  - Manual tag addition
    |
    v
Clicks "Save" or "Save & Close Tab"
    |
    v
Content stored locally (no internet required for basic capture)
```

### 3.2 Organization and Tagging

#### 3.2.1 Multi-Dimensional Tag System

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `project:*` | What you are working on | `project:ai-development`, `project:research-tracking` |
| `person:*` | Who it relates to | `person:family-member-a`, `person:colleague` |
| `type:*` | Content type | `type:tutorial`, `type:reference`, `type:article` |
| `context:*` | When/where needed | `context:doctor-appointment`, `context:travel` |
| `status:*` | Workflow state | `status:to-read`, `status:in-progress`, `status:completed` |
| `source:*` | Auto-generated origin | `source:claude`, `source:gmail`, `source:nytimes` |

#### 3.2.2 Tag Features

- Multiple tags per item (unlimited)
- Tag metadata (color, icon, related tags, parent tag, item count)
- Tag relationships and visual graph
- AI-powered tag suggestions with confidence levels
- Smart collections: untagged, recent, popular, orphaned

### 3.3 Search and Retrieval

**Basic Search:** Full-text, by tags, date range, source type

**Advanced Search:** Boolean operators, tag combinations, exclude filters

**Natural Language Queries:** AI interprets and executes queries

**Cross-Reference Search:** Find related items, timeline view

**Search Interface Views:** List, Calendar, Tag Graph, Project

### 3.4 Daily Brief Generation

AI-generated morning summary with configurable sections:

```
TOP PRIORITY
- Overdue tasks and today's deadlines

TODAY'S CALENDAR
- Upcoming meetings and events with related captured information

IMPORTANT EMAILS
- Flagged emails matching important projects and contacts

RELEVANT RESEARCH
- Recently captured items for active projects
- Unused resources ready to use

SUGGESTIONS
- AI-generated recommendations based on usage patterns
```

**Example Daily Brief (generic):**

```
Good morning! Here is your brief for Monday, March 10.

TOP PRIORITY
- Project proposal draft is due today
- Follow up on vendor quote from last week

TODAY'S CALENDAR
- 10:00 AM - Team standup (Conference Room B)
  Related: You saved 3 articles on the sprint topic last week
- 2:00 PM - Dentist appointment
  Related: Insurance card photo captured on Feb 15
- 6:00 PM - Child's soccer practice

IMPORTANT EMAILS
- Reply pending from supplier about order #4521
- Newsletter from industry publication (matches project:research)

RELEVANT RESEARCH
- 5 unread items tagged project:home-renovation
- 2 unused references for project:research-tracking

SUGGESTIONS
- You have 12 items tagged status:to-read older than 30 days
- Consider archiving 8 completed items from last month
```

**Delivery:** Email at configurable time, web dashboard, mobile notification (future)

---

## 4. Technical Architecture

### 4.1 Platform Strategy

**Browser Extension (Primary)**
- Target: Chrome, Firefox (WebExtensions API)
- Safari: Future consideration
- Advantages: Rich functionality, offline support, keyboard shortcuts, page injection, browser APIs

**Bookmarklet (Alternative)**
- Works on all browsers including Safari
- Limited functionality, requires backend from day one

**Backend API (Future Phase)**
- Required for: cloud sync, email/calendar integration, AI processing, daily briefs
- Not required for MVP

### 4.2 Data Storage

| Phase | Storage | Notes |
|-------|---------|-------|
| MVP | IndexedDB | Browser-native, offline capable, handles nested data |
| Phase 2+ | PostgreSQL | Cloud sync, vector embeddings for semantic search |

### 4.3 AI Integration

- Tag suggestion on capture
- Natural language search
- Daily brief generation
- Tag relationship discovery
- Content summarization
- Duplicate detection
- **Implementation:** Claude API, OpenAI embeddings, local processing where possible

### 4.4 Data Model

**Captured Item:**

```javascript
{
  id: "uuid",
  url: "https://example.com/article",
  title: "Article Title",
  notes: "User's notes...",
  screenshot: "base64-encoded-image",
  fullText: "Extracted article text...",
  capturedAt: "2025-01-18T10:30:00Z",
  lastAccessedAt: "2025-01-18T14:22:00Z",
  accessCount: 3,
  tags: ["project:ai-development", "type:tutorial", "source:anthropic"],
  aiSuggestedTags: ["project:ai-music"],
  metadata: {
    author: "Jane Smith",
    publishDate: "2025-01-15",
    readingTime: "5 min",
    domain: "example.com"
  },
  source: {
    type: "webpage",
    platform: "chrome",
    tabTitle: "Original tab title"
  }
}
```

**Tag Definition:**

```javascript
{
  id: "project:ai-development",
  type: "project",
  displayName: "AI Development",
  color: "#3b82f6",
  icon: "robot",
  relatedTags: ["project:ai-music", "project:ai-coding"],
  parentTag: "project:ai",
  childTags: ["project:ai-coding", "project:ai-music"],
  createdAt: "2025-01-01T00:00:00Z",
  lastUsedAt: "2025-01-18T10:30:00Z",
  itemCount: 47,
  metadata: {
    status: "active",
    priority: "high",
    description: "Learning and implementing AI development tools"
  }
}
```

---

## 5. User Interface

### 5.1 Extension Popup (Quick Capture)

```
+-------------------------------------------+
| Capture: "Article Title"                  |
+-------------------------------------------+
| URL: https://example.com/article          |
| Source: Chrome Tab                        |
+-------------------------------------------+
| Notes:                                    |
| +---------------------------------------+ |
| | User's thoughts and highlights...     | |
| +---------------------------------------+ |
+-------------------------------------------+
| Tags:                                     |
| [x] project:ai-development               |
| [x] type:tutorial                         |
| [+] Add tag...                            |
|                                           |
| AI Suggests:                              |
| [+] project:ai-music (related project)   |
+-------------------------------------------+
| [Save]  [Save & Close Tab]  [Cancel]     |
+-------------------------------------------+
```

### 5.2 Dashboard (Main Interface)

- **Top Navigation:** Search bar, filter dropdowns, view toggles, settings
- **Left Sidebar:** Tag categories (collapsible), smart collections
- **Main Content:** Item list/grid with previews, sorting, bulk actions
- **Item Detail:** Full content/screenshot, metadata, edit notes/tags, related items, history

### 5.3 Tag Manager

- Visual hierarchy with item counts and last used dates
- Tag detail view with all items, related tags, usage statistics
- Tag graph visualization: node size = item count, edge thickness = relationship strength

---

## 6. Development Phases

| Phase | Timeline | Focus | Key Features |
|-------|----------|-------|--------------|
| 1 (MVP) | Weeks 1-4 | Basic capture | Chrome extension, manual tagging, IndexedDB, list view, search, export |
| 2 | Weeks 5-8 | AI Enhancement | AI tag suggestions, NL search, duplicate detection, Firefox support |
| 3 | Weeks 9-12 | Integration | Google Calendar/Gmail read-only, daily briefs, cloud sync |
| 4 | Month 4+ | Polish and Scale | Proton Mail, Evernote import, AI chat parsing, team features, API |

---

## 7. Success Metrics

### Usage Metrics

| Metric | Target |
|--------|--------|
| Captures per day | 10+ |
| Searches per week | 20+ |
| AI tag suggestion acceptance rate | >60% |
| Daily brief open rate | >80% |

### Quality Metrics

| Metric | Target |
|--------|--------|
| Retrieval success rate | High |
| Tag coverage | Comprehensive |
| Duplicate rate | Low |
| Data loss incidents | Zero |

### Engagement Metrics

- Daily active usage
- Meaningful session duration
- High return rate

---

## 8. Privacy and Security

- All data stored locally by default
- Cloud sync is optional and user-controlled
- No data sold or shared with third parties
- Full export and delete capability
- AI processing via API calls (can be disabled)
- HTTPS for all network communication
- OAuth for third-party integrations
- No password storage

---

## 9. Competitive Landscape

| Competitor | Strengths | Weaknesses | ThoughtBox Opportunity |
|-----------|-----------|------------|----------------------|
| Notion/Coda | Rich workspace | Manual organization, no native integrations | Passive, intelligent capture |
| Mem.ai | AI-powered | Closed ecosystem, no multi-source | Open architecture, privacy-first |
| Evernote | Good web clipper | Weak AI, siloed data | AI organization layer |
| Raindrop.io | Great bookmarking | Weak AI, no synthesis | AI layer on bookmarking |
| Obsidian | Local-first, plugins | No web capture | Browser-native experience |

### Unique Value Propositions

- Only tool handling multiple AI chat platforms
- Only tool connecting Proton and Google ecosystems
- Task-centric organization
- Privacy-first with local storage
- Multi-dimensional tagging versus simple folders

---

## Appendix A: Example Workflows

### Research Tracking Workflow

1. Discover a useful article or fact during web browsing
2. Capture it with ThoughtBox, adding tags like `project:research`, `status:unused`
3. When incorporating the resource into work, update status to `status:used`
4. Before starting new research, search ThoughtBox for `project:research AND status:unused` to see what is already available
5. Daily brief highlights unused resources ready for the current project

### Family Activity Planning Workflow

1. Receive email about an upcoming event -- capture the email link with `context:family-events`
2. Find a related web page with details -- capture with same tag
3. Calendar event is created -- linked in ThoughtBox
4. Before the event, search by tag to see all related information in one place
5. Daily brief surfaces relevant items on the day of the event

### AI Development Learning Workflow

1. Ask Claude about a coding pattern -- capture the conversation with `project:ai-development`, `type:tutorial`
2. Find a related tutorial on the web -- capture with same project tag
3. Later, search "what did I learn about embeddings?" using natural language search
4. ThoughtBox returns results from both the AI conversation and the web tutorial
5. Tag graph shows connections between related AI development topics

---

## Appendix B: Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| IndexedDB storage limits | Medium | High | Monitor usage, implement cleanup and archive |
| Claude API rate limits | Medium | Medium | Queue requests, batch processing, cache results |
| Browser extension API changes | Low | High | Abstract APIs, monitor deprecation notices |
| Screenshot storage bloat | High | Medium | Compress images, offer link-only capture mode |
| Feature creep delays MVP | High | Medium | Strict Phase 1 scope enforcement |

---

## Appendix C: Data Migration Strategy

### Import Priority

1. Browser bookmarks
2. Evernote (ENEX format)
3. Delicious / Pinboard
4. Pocket / Instapaper
5. Notion

### Export Formats

- JSON (full fidelity)
- Markdown
- CSV
- HTML
- OPML

---

## Appendix D: Accessibility Requirements

- Full keyboard navigation with configurable shortcuts
- ARIA labels and screen reader support
- Minimum 4.5:1 contrast ratio
- Color-independent status indicators
- Large click targets (44x44px minimum)
- Dark mode support

---

## Appendix E: Backup and Recovery

- Daily auto-export to JSON
- 30-day trash retention for deleted items
- Transaction-based writes via Dexie.js for data integrity

---

## Appendix F: Sync and Conflict Resolution (Phase 2+)

| Field Type | Strategy |
|-----------|----------|
| Simple fields (title, notes) | Last-write-wins |
| Additive fields (tags) | Union merge |
| Complex conflicts | Present conflict UI for manual resolution |

- Offline queue with exponential backoff retry
- All sync operations are idempotent
