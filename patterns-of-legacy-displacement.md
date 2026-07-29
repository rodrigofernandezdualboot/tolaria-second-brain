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

> **Verification status (29 Jul 2026):** the previous version of this note was written from prior knowledge and contained errors — see the correction log at the bottom. This version is checked against the live article. Four pattern pages were read in full: Critical Aggregator, Divert the Flow, Extract Product Lines, Revert to Source. The other four are summarized from cross-references within those pages, not from their own pages — treat those four as provisional.

## Core idea

Replacing a legacy system is risky because the old system encodes years of accumulated business rules and behavior that nobody fully understands. Rather than a single "big bang" rewrite, the authors advocate **incremental displacement**: carve the legacy system into pieces and replace them one at a time, keeping the whole running throughout.

The work is framed as three intertwined activities that repeat:

1. **Understand the outcomes** the system delivers (not just its current implementation).
2. **Decide how to break the problem into smaller parts** you can displace independently.
3. **Successfully deliver** each part into production, learning as you go.

## The eight published patterns

Per the article's own navigation:

| Pattern | Strapline | Read in full |
|---|---|---|
| Critical Aggregator | Combine data from different parts of the business to support making critical decisions | ✅ |
| Divert the Flow | First divert cross-organization activities away from legacy | ✅ |
| Event Interception | Capture and redirect events flowing into legacy | — |
| Extract Product Lines | Identify and separate systems by product line | ✅ |
| Feature Parity | Replicate needed features, not the whole legacy feature set | — |
| Legacy Mimic | New system behaves toward the outside world as legacy did | — |
| Revert to Source | Identify the originating source of data and integrate to that | ✅ |
| Transitional Architecture | Scaffolding built for the migration and expected to be discarded | — |

`[ASSUMPTION]` **Extract Value Streams** is linked from the body of *Extract Product Lines* but does not appear in the published pattern navigation — it looks like a stub or planned page rather than a published pattern. Do not cite it as one.

### Critical Aggregator — and its anti-pattern

A software component that knows which systems to visit to extract information, how to relate data from different sources, and the business logic to aggregate it. Delivered as reports, dashboards, or data feeds.

Done well with encapsulation and separation of concerns, this is unremarkable. The problem is the **Invasive Critical Aggregator** — the legacy manifestation, where the aggregator reaches directly into sub-components to fetch data with no encapsulation. Characteristics:

- Processing and data-access code intermingled; usually written in batch data-processing languages.
- Any change to source data structure immediately breaks processing and outputs. The common response is to **freeze source data formats or impose change control** — which spreads the freeze to the upstream systems.
- Scales poorly: no encapsulation means optimization and parallelism are hard, so execution time grows with data volume and you end up vertically scaling the whole system.
- Susceptible to timing issues. There's usually an undocumented "safe time window" during which source data must be current and stable, enforced by convention rather than by the system.
- Outputs **bloat** over time, because extending an existing report is easier than creating a new one. This makes replacement harder — you first have to decompose who actually uses which output.
- Typically ends up frozen, because the business risk of wrong data is too high.

Two displacement approaches: build a new implementation via **Divert the Flow** (often with **Revert to Source**), or leave it in place and use **Legacy Mimic** to keep feeding it. The second is more common; a new implementation is needed eventually either way.

### Divert the Flow

Start the displacement initiative by building a **new implementation of the Critical Aggregator, decoupled as far as possible from the upstream systems** that source its data. Once it's live, disable the legacy implementation — which frees you to change or relocate the upstream sources.

The alternative is to leave the aggregator until last, which forces you to keep legacy fed via Legacy Mimic while you displace upstream, leaving new components coupled to legacy data structures and update frequencies, and leaving a large user population with no improvement until near the end.

Steps the article calls out:

- **Map data sources** to the *ultimate* upstream source, not the intermediate one. Capture which upstream data legacy currently discards — that's often available value.
- **Understand user requirements.** Aggregators typically have a very mixed user base, and this is a classic place where Feature Parity produces bloated rebuilt reports nobody needs. A smaller set of targeted reports is often better.
- **Capture how outputs are produced**, accepting diminishing returns. Sometimes the legacy code is the only documentation, and reverse-engineering costs more than redefining from first principles.
- **Delivery and testing:** iterate report by report to production, use parallel running with reconciliation. Expect the new outputs *not* to match — legacy reports commonly contain undiscovered bugs. Inject known data and validate outputs; do this for the new system at minimum.
- Hunt for **"off system" workarounds** — manual spreadsheet manipulation between the legacy output and what leadership actually sees. Often nobody admits this exists.
- **Go live** in staged cohorts; keep parallel running and reconciliation alerting for a period.
- Fix data issues **in the upstream systems** rather than reintroducing legacy workarounds into the new solution.

Also flagged: if the data warehouse is itself legacy, you can Divert the Flow away from it — but feeding a legacy DWH an identical feed from new systems re-couples you to its formats and update frequencies. Organizations have replaced most of an estate and still run the business on stale data because of this.

### Extract Product Lines

Many systems serve multiple logical products from one physical system, usually justified by reuse. Superficially the products look similar; in the detail they differ a lot. Over time the single system becomes over-generic, evolving to handle every combination.

The testing argument is the sharp one: *n* products with *n* changes per product implies factorial test combinations. The authors note that systems of this type they've encountered "have had very little in the way of automated test coverage, instead relying on huge, often manual regression suites" — because testing that many code paths isn't possible.

So the problem is economic: maintaining a system per product can beat maintaining one generic system, because you avoid the combinatorial explosion. It's a trade-off — not red trousers vs. black trousers, but off-the-peg vs. made-to-measure.

How it works:

1. **Identify the product lines** — this forms the thin slice. Map existing capabilities to the new product, looking through several lenses (capability, data, process, users).
2. **Identify shared capabilities.** Value *use* over *reuse* — limit shared capabilities as much as possible.
3. **Choose who goes first.** The article's heuristic: take **the second riskiest product line**. Counter-intuitive, but you want something meaty enough to hold business attention and funding, without the business failing if it goes wrong.
4. **Identify target architecture** for the thin slice only — not a big-bang build of all products.
5. **Identify technical migration strategy** — ForkByUrl, ForkingOnIngress/MessageRouter, and expect a Transitional Architecture.

Use when product lines would benefit from independent work (team autonomy, no merge hell or long regression cycles), have different non-functional characteristics/SLOs, or have different rates of change.

The insurance worked example: Vehicle, Home, Life, Pet on one 3-tier quoting engine. Ranked by revenue as Vehicle > Home > Life > Pet, and by customer numbers in reverse. **Home was chosen first** — balancing risk against enough revenue to matter. Delivery was event-driven over RabbitMQ, with an adapter turning new-world events into the star schema the old data warehouse required. Cutover was beta opt-in, then 1% → 5% → 10% of traffic.

### Revert to Source

Once a system has been integrated into the legacy monolith, everyone reaches it *through* legacy rather than repeating the integration. Repeat that and legacy becomes the single integration point and de facto upstream source for business processes — and the originating system gets forgotten.

By tracing data back "beyond" the legacy estate, you can integrate directly to the true source. This reduces legacy dependency early and usually improves data quality and timeliness.

- Produce a **data flow map** that doesn't stop at legacy but digs into the underlying integration points. Trace upstream (from business process back through legacy — needs the people who built or support it) or downstream (from an integration point forward to the capabilities it feeds, like dye in a river). Downstream tracing is often neglected and is the better option when legacy knowledge has been lost.
- If new direct integration isn't possible, use **Event Interception** to copy the data flow, as far upstream as possible.
- **When to use it:** extracting a capability whose data ultimately comes from an integration point hiding behind legacy, and where the data passes through legacy broadly unchanged. The main things legacy does to it are **loss of data** (fields with no legacy representation) and **loss of timeliness** (batch imports, fixed safe-update windows).
- Combine with parallel running and reconciliation to prove legacy isn't altering the data.
- Upstream and downstream flows can be split — take reads direct from source while writes continue through legacy.
- Check the source system's cross-functional constraints first; don't overload it or discover it isn't reliable enough.

The retail example: a website sourcing stock from legacy via overnight batch could only sell warehouse stock, because shop stock moved during the day. A new inventory component integrated directly with in-store till systems over a network the business had already built for card payments, unlocking in-store stock for online sale — plus much more timely sales data.

### The remaining four (from cross-references only)

- **Event Interception** — capture events/updates flowing into legacy and redirect or duplicate them to the new component, enabling incremental redirection of behavior. Used as the feed mechanism for Divert the Flow when Revert to Source isn't available.
- **Feature Parity** — replicate only the *needed* features. Existing features are a starting point for understanding, not a specification to copy. The article repeatedly warns that parity thinking produces bloated rebuilds of things users don't need.
- **Legacy Mimic** — the new system behaves toward the outside world and toward remaining legacy exactly as legacy did, so surrounding systems can't tell who is serving them.
- **Transitional Architecture** — adapters, routers, and sync jobs built specifically to support the migration, expected to be thrown away. Plan and budget for its removal.

## Key takeaways

- Displace, don't rewrite: incremental replacement beats big bang.
- Chase needed outcomes, not literal feature parity.
- Slice vertically — by product line — not by technical layer.
- Pick the **second riskiest** slice first, not the safest.
- Consider diverting the aggregator/reporting flow *early* rather than leaving it until last; it's what unfreezes the upstream systems.
- Expect new outputs to disagree with legacy, because legacy has undiscovered bugs. Parallel running with reconciliation is how you find out which is right.
- Expect and plan to discard transitional architecture.

## Applied

- [[park-road-granite-platform-takeover-roadmap]] — ProWeb's Power BI-on-OLTP reporting is a textbook Invasive Critical Aggregator; Granite's four solution lines are the Extract Product Lines axis.

## Correction log

Errors in the previous version of this note, found on verification:

- **"Asset Capture"** — listed as a pattern. Not found in the article or its navigation. Removed.
- **"Feature Access Variations"** — listed as a pattern. Not found. Removed.
- **"Extract Value Streams"** — presented as a published pattern. It is linked from *Extract Product Lines* but is not in the published pattern list; the actual pattern is **Extract Product Lines**. Corrected.
- **Critical Aggregator** and **Revert to Source** — both omitted entirely. Added.
- The "pick who goes first" guidance was absent, and it is counter-intuitive enough to matter (second riskiest, not safest). Added.
