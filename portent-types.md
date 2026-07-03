---
type: Note
related_to: "[[tolaria]]"
_organized: true
_favorite: true
_favorite_index: 1
---

# Portent Types

Reference for the default type vocabulary defined by [Portent](https://portent.md), the open specification and template Tolaria uses as its best-practice model. Use this as the starting vocabulary when creating future entities in this vault.

Portent favors **convention over configuration**. Instead of "where should this go?", it asks:

- What is this?
- What is it useful for?
- Is it captured, organized, or archived?

## The eight default types

Portent defines eight types split into two groups.

### PORT — actionable

- **Project** — a defined effort with an outcome. _Example: "Launch v2 of the client portal."_
- **Operation** — ongoing work or a recurring process. _Example: "Weekly newsletter production."_
- **Responsibility** — an area of standing accountability. _Example: "Team hiring and onboarding."_
- **Task** — a single actionable item. _Example: "Draft the Q3 budget proposal."_

### ENTP — non-actionable knowledge records

- **Event** — something that happened or is scheduled at a point in time. _Example: "Kickoff meeting — 2026-07-10."_
- **Note** — a general knowledge record. _Example: "Notes on the Portent type model."_
- **Topic** — a subject or theme that ties notes together. _Example: "Knowledge management."_
- **Person** — an individual. _Example: "Jane Doe, product lead."_

These defaults cover the common shape of personal and work knowledge with almost no setup. Add custom types later, but start from this vocabulary first.

## Relationships

Portent models knowledge as a graph with two default relationships:

- `belongs_to` — primary ownership, composition, or context.
- `related_to` — a looser semantic connection.

Both live in YAML frontmatter and point to other notes with wikilinks, keeping the graph portable, searchable, and readable outside the app.

## Lifecycle

Portent separates capture from organization:

1. **Capture** information quickly so it is not lost (Inbox).
2. **Organize** it by assigning a type and useful relationships.
3. **Archive** it when it has served its purpose.

Tolaria supports this directly: the Inbox holds captured notes, organizing marks a note ready for normal views, and archiving hides obsolete notes from active surfaces while keeping them available.

## See also

- Bundled Tolaria docs: `templates/portent.md`, `concepts/types.md`
- [portent.md](https://portent.md) for the full spec
- Template vault: [refactoringhq/portent-vault-template](https://github.com/refactoringhq/portent-vault-template)
