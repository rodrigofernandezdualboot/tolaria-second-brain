---
type: Note
status: Active
url: https://www.martinfowler.com/articles/patterns-legacy-displacement/
author: Ian Cartwright, Rob Horn, James Lewis
source: martinfowler.com
_organized: true
belongs_to: "[[software-architecture]]"
---

# Patterns of Legacy Displacement

Reference summary of the martinfowler.com article **Patterns of Legacy Displacement** by Ian Cartwright, Rob Horn, and James Lewis.

> Note: captured from prior knowledge of the article, not a live fetch. Verify details against the [source](https://www.martinfowler.com/articles/patterns-legacy-displacement/) before relying on specifics.

## Core idea

Replacing a legacy system is risky because the old system encodes years of accumulated business rules and behavior that nobody fully understands. Rather than a single "big bang" rewrite, the authors advocate **incremental displacement**: carve the legacy system into pieces and replace them one at a time, keeping the whole running throughout.

The work is framed as three intertwined activities that repeat:

1. **Understand the outcomes** the system delivers (not just its current implementation).
2. **Decide how to break the problem into smaller parts** you can displace independently.
3. **Successfully deliver** each part into production, learning as you go.

## The patterns

### Understanding / decomposing

- **Feature Parity** — Aim to replicate only the *needed* features, not the whole legacy feature set. Existing features are a starting point for understanding, not a specification to copy verbatim. Beware "feature parity" becoming an excuse to rebuild everything.
- **Extract Value Streams** — Slice the system along end-to-end flows of business value (a vertical slice a customer cares about) rather than along technical layers. Each value stream becomes a candidate for independent displacement.
- **Feature Access Variations** — Different consumers use features in different ways; identify these variations so you can peel off simpler cases first and defer the complex ones.

### Delivering / migrating

- **Legacy Mimic** — The new system behaves toward the outside world (and toward the remaining legacy) exactly as the legacy did, so surrounding systems can't tell which is serving them during transition.
- **Event Interception** — Capture events/updates flowing into the legacy system and redirect (or duplicate) them to the new component, allowing incremental redirection of behavior.
- **Asset Capture** — Move data/functionality ("assets") into the new system while the legacy continues to run, gradually making the new system the source of truth.
- **Transitional Architecture** — Software and scaffolding built specifically to support the migration (adapters, routers, sync jobs) that is expected to be thrown away once displacement completes. Plan for its removal.
- **Divert the Flow** — Reroute the flow of a process from legacy to the new implementation, often behind the Legacy Mimic, so traffic shifts incrementally and reversibly.

## Why it matters / when to apply

- Reduces the enormous risk of big-bang rewrites by delivering value continuously and keeping a working system at all times.
- Forces genuine understanding of business outcomes instead of blindly copying legacy behavior.
- Enables reversibility — if a slice fails, you can fall back to the legacy path.
- Transitional architecture is a deliberate, temporary cost — budget for building *and* removing it.

## Key takeaways

- Displace, don't rewrite: incremental replacement beats big bang.
- Slice by value stream (vertical), not by technical layer.
- Chase needed outcomes, not literal feature parity.
- Use Legacy Mimic + Event Interception + Divert the Flow to move traffic safely and reversibly.
- Expect and plan to discard transitional architecture.
