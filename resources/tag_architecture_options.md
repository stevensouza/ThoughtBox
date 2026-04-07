# Tag Architecture: Design Options

**Status:** Open design question  
**Date:** April 7, 2026  
**Context:** ThoughtBox needs a tagging system that balances flexible flat tagging with the navigability of hierarchies. This document captures three architectural options explored during brainstorming.

## Background

There are three established patterns for organizing tagged content in knowledge management systems:

- **Flat tags** (Gmail labels, old Delicious): every tag is equal, no structure
- **Hierarchical taxonomy** (file systems, Evernote notebooks): strict tree, one parent per tag
- **Faceted classification** (e-commerce filters, library science): multiple independent dimensions/axes

ThoughtBox's earlier brainstorming introduced prefixed tag categories (`project:*`, `person:*`, `status:*`, etc.) and a Tag DAG concept where tags can have multiple parents. The question is how these ideas fit together.

---

## Option A: Facets Are Special, Tags Are Flat Within Them

The prefix IS the facet. Each facet is a hard-coded dimension with its own behavior.

```
FACETS (built-in, each gets its own UI):
  project: → has its own dashboard, sub-projects, deadlines
  person:  → has its own contact-card-style view
  status:  → has its own kanban/workflow view
  type:    → mostly for filtering
  source:  → auto-generated, mostly for filtering

Tags within a facet can have hierarchy:
  project:ai → project:ai-development → project:ai-music

But cross-facet hierarchy is not allowed:
  project:tab-manager CANNOT be parent of person:cici
```

### Pros
- Simplest mental model for users
- Each facet type can have purpose-built UI without abstraction overhead
- Familiar pattern (closest to how most apps work)

### Cons
- Adding a new facet type requires code changes
- Cross-cutting concerns are awkward (a bug that's both a project subtask and a person's responsibility)
- The DAG concept doesn't apply across facets, only within them
- Rigid: if you realize a "topic" tag should actually be a "project," migration is messy

---

## Option B: Everything Is a Tag, Some Have a `kind` Property

No facets. Every tag is a node in one universal DAG. A `kind` property is just metadata.

```
ALL TAGS ARE EQUAL IN THE DAG:
  tab-manager (kind: project)
  bug23       (kind: topic)
  cici        (kind: person)
  urgent      (kind: priority)

Any tag can parent any other tag:
  tab-manager → bug23 → backend   (valid)
  cici → ai-music                 (valid)
  urgent → bug23                  (valid)

The `kind` controls:
  - Default icon/color
  - Which optional fields appear (deadline for projects, email for people)
  - Which "views" the tag appears in
```

### Pros
- Maximum flexibility: any tag can relate to any other tag
- Adding a new kind is just adding an enum value, no schema change
- The full DAG works as designed (wildcard search, multiple parents, drop-order primary)
- A tag's kind can change without breaking relationships

### Cons
- Can become confusing: "why is a person a child of a status?"
- UI needs guardrails to prevent nonsensical hierarchies
- "Everything is everything" can lead to organizational chaos without discipline
- Harder to build purpose-built views when the underlying model is so generic

---

## Option C: Hybrid (Recommended)

Tags are the universal primitive. `kind` is optional metadata that drives views and suggested fields but does NOT constrain the DAG.

### Core Principle
**Views are lenses over the same data, not separate containers.**

### Data Model

```javascript
{
  id: "uuid",
  name: "bug23",
  kind: "topic",          // optional: project|person|status|context|type|source|topic|null
  displayName: "Bug #23",
  color: "#ef4444",
  icon: null,             // inherits from kind default if null
  parents: [
    { tagId: "tab-manager", isPrimary: true,  createdAt: "..." },
    { tagId: "backend",     isPrimary: false, createdAt: "..." },
    { tagId: "urgent",      isPrimary: false, createdAt: "..." }
  ],
  metadata: {             // kind determines suggested keys, but any key is allowed
    status: "in-progress",
    assignee: "steve"
  },
  createdAt: "...",
  itemCount: 12
}
```

### How `kind` Works

- **Optional:** A tag with no kind is just a tag. No restrictions.
- **Controls views:** "Project Dashboard" is a filtered view (`kind=project`), not special code.
- **Controls suggested metadata:** Setting `kind=project` offers deadline/status fields. Setting `kind=person` offers email/role fields. But any tag can have any metadata key.
- **Does NOT constrain hierarchy:** A person-kind tag CAN be a child of a project-kind tag. The DAG is kind-agnostic.
- **Changeable:** Switching a tag's kind from `topic` to `project` preserves all DAG relationships.

### Views Are Queries

| View | Filter | Extra UI |
|------|--------|----------|
| Projects | `kind=project` | Deadline, status badge, sub-project tree |
| People | `kind=person` | Email, related items, activity timeline |
| Kanban | `kind=status` | Drag items between status columns |
| Everything | (no filter) | Full DAG explorer, graph view |
| Custom | User-defined | User picks metadata fields to display |

Adding a new kind (e.g., `location`) means adding one enum value and optionally defining a view template. Zero schema changes required.

### Pros
- Full DAG flexibility (all prior wildcard/multi-parent work applies)
- Purpose-built views for common kinds (project, person) without hard-coding them into the data model
- New kinds are trivial to add
- Changing a tag's kind is non-destructive
- The `project:*` prefix syntax becomes search sugar, not structural constraint
- Scales from simple (flat tags, no kinds) to sophisticated (full DAG with custom views)

### Cons
- Slightly more complex mental model than Option A
- Need to design a view-definition system (though can start with hard-coded views and generalize later)
- Risk of over-engineering if the view system gets too abstract too early

---

## Comparison Summary

| Concern | Option A (Facets) | Option B (Pure DAG) | Option C (Hybrid) |
|---------|-------------------|--------------------|--------------------|
| Adding new tag category | Code change | Enum value | Enum value |
| Cross-category hierarchy | Not allowed | Unrestricted | Unrestricted |
| Purpose-built screens | Hard-coded per facet | Query-driven | Query-driven with kind hints |
| Changing a tag's category | Messy migration | Change kind property | Change kind property |
| DAG/wildcard search | Within-facet only | Full | Full |
| Implementation complexity | Lowest | Medium | Medium (can start simple) |
| Risk of user confusion | Low | Highest | Medium (views provide structure) |

---

## Implementation Strategy (if Option C)

**Phase 1:** Flat tags with optional `kind` property. No hierarchy. Hard-coded views for `project` and `person` kinds. Search by tag name and kind prefix.

**Phase 2:** Single-parent hierarchy. Tags can have one parent. Primary views show tree structure. Wildcard search (`*`, `?`).

**Phase 3:** Full DAG. Multiple parents with `isPrimary` ordering. `**` glob search. Custom view definitions.

This mirrors the existing phased approach in the requirements doc and avoids building DAG complexity before validating that basic tagging works well.

---

## Open Questions

1. When you look at a "Project Dashboard," what info should appear beyond "all items tagged with this project"? (Deadlines? Status rollup? Sub-project tree?)
2. Will you create new kinds frequently, or is the current set (project, person, status, type, context, source) fairly stable?
3. Should cross-kind parenting be unrestricted, or are there combinations that feel wrong? (e.g., status as child of person)
4. Should the `kind` enum be user-extensible, or is a fixed set with a `null`/generic fallback sufficient?
