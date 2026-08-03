---
type: Note
related_to: "[[artsavl]]"
status: Active
_width: wide
---

# ArtsAVL — Effort Estimation & Budget Gap

Run 2026-08-03 through the project-assessor workflow (steps 1–6 complete, step 7 verdict pending). Source of truth for hours is `engagements/artsavl-creative-portal/estimation/output/project-assessment.md`. Context: [[artsavl-technical-assessment]] · [[artsavl-risk-register]] · [[artsavl-solution-architecture]] · [[artsavl-client-questions]] · [[artsavl-assessment-findings-vs-initial-read]]

## Headline

**The scope does not fit the budget, and it is not close.**

| | Hours |
|---|---|
| AI-calibrated total | **2,298** |
| Traditional baseline | 2,538 |
| PERT expected *(O+4M+P)/6* | 2,385 |
| **With 15% management reserve** | **≈ 2,743** |

Three-point: optimistic **2,020** · most likely **2,298** · pessimistic **3,100**.

Rate sensitivity against the $150–250K ceiling (no Dualboot rate card used — apply the real blended rate):

| Blended rate | Cost of 2,743h | vs $250K |
|---|---|---|
| $95/h | ~$261,000 | over |
| $125/h | ~$343,000 | **37% over** |
| $150/h | ~$411,000 | **65% over** |

Inverted: $250K buys roughly **1,650–2,000 hours**. All-six-thin needs **~2,750**. **Gap ≈ 700–1,100 hours, or 30–40% of scope.**

This is risk **B2** moving from an argument to an arithmetic result. It was scored Critical before any numbers existed; the WBS confirms it independently.

## Basis

`estimationScenario: AI_CALIBRATED` · `stackFamiliarity: Confirmed` (velocity rates apply without modifier) · **FTE 3.5** (1 TL + 2 devs + PM + designer) · AI toolchain preset confirmed (Cursor, Claude Code, Figma MCP).

Anti-overoptimism check passed: adjusted total is 90.5% of base (70% floor), no Epic reduced more than 30%.

**Confirmed gate decisions:** all six goals at first-useful-version · WCAG **2.2 AA** via audit + top-tier remediation only · architecture §9 Goal-4 exclusions as written · prior 22-risk register carried forward re-scored.

## Where the hours are

| Epic | AI-adj. | Note |
|---|---|---|
| E-09 UX + accessibility remediation | **324** | Design 130h from zero — no UI materials exist at all (`designCompleteness: Wireframe` ×1.0). Least-specified Epic in the plan |
| E-04 Tenancy foundation | **304** | Largest single item. Data migration across 9 entity groups is ~146h once validation, dry-run and rollback are included. Gated on the E-02 spike |
| E-11 QA & testing | 270 | ~18% of dev hours + E2E + UAT; includes cross-tenant isolation assertions added to the inherited 2,000-example suite |
| E-02 Assessment, discovery, audits, spike, validation | 258 | Codebase analysis across 8 domain modules; 3 peer-council validation conversations; accessibility audit; tenancy spike |
| E-05 Per-partner config, branding, one partner live | 131 | CSS-custom-property branding, per-tenant taxonomy, custom domain onboarding |
| E-06 Rollups & reporting | 130 | Rollup tables, configurable staff dashboard, per-tenant scoping |
| E-08 Revenue workflows | 116 | **Self-service advertising is a 60h new build, not a refinement** — no member-side ad route exists |
| E-07 Communications & integrations | 103 | Newsletter sync priced as a *complex* integration because the vendor is still unidentified |
| E-10 Syndication | 85 | iCal, JSON Feed, iframe widget, per-consumer keys |
| E-03 Discoverability & structured data | 58 | Serves Goals 1 and 5 with one body of work |
| E-01 Transition & continuity | 47 | Verified access across 11 services, live test transaction and send, Ruby 4.0.6 patch, dormant-credential rotation |
| E-12 PM & overhead | **472** | PM at 15%, TL code review 144h, release management. AI velocity = 0% by rule |

Two items worth knowing before any client conversation: the tenancy **data migration** is far heavier than "add a tenant column" implies, and **advertising self-service is net-new**.

## What actually fits

Deferring the Goal-4 Epics (E-04 + E-05 = 435h) and syndication (E-10 = 85h) gives ~1,778h before reserve, ~2,045h with — about $256K at $125/h, still marginally over. Trimming E-09 to the accessibility audit plus the highest-value flows brings it inside $250K.

**The shape that fits:** continuity · assessment and bounded discovery · discoverability · reporting · revenue workflows · accessibility audit plus top-tier remediation · **a tenancy architecture decision document rather than a tenancy build** — with tenancy implementation, partner onboarding and syndication sequenced into a defined, costed second phase.

That is the honest form of the proposal RFP §11 explicitly invites ("recommendations are welcome" where a firm believes a different scope better fits the budget).

## Assumptions the number rests on

Seven, all flagged rather than resolved. The two that matter most:

- **[A-01]** RFP §4's account of code quality (2,000+ specs, 154 E2E tests, maintained docs) is accurate. Weakened by the Linxup/Rastrac finding, which shows §4 was published partly unverified.
- **[A-02]** The account model decomposes into a tenant boundary as "accounts serve as the ownership and billing root" implies. **Unverifiable without code, and E-04 is the largest Epic.**

Also: **[A-03]** reporting and communications exist in some admin form and are therefore estimated as *enhancement, not net-new* — RFP §4 says they exist but **the live crawl found no admin route for any of them**. If they turn out minimal, Goal 2 grows. [A-05] no Stripe Connect in v1. [A-06] one partner onboarded. [A-07] client supplies a product owner at 8–10 hrs/week — this drives *duration*, not hours.

## Open decisions before the verdict closes

1. **Scenario headline** — carry Most Likely (2,298) or PERT expected (2,385), and hold the reserve at 15%?
2. **The real question.** All-six-thin was chosen knowing the risk; the estimate now sizes it at a third of scope. Either the bid changes, or bidding over budget becomes a deliberate, argued position. Both are defensible against a client who invited rescoping — but it is a commercial call, not an analytical one.

**My read: CONDITIONAL GO, with rescoping as the condition.**
