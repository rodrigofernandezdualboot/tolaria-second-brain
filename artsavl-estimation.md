---
type: Note
related_to: "[[artsavl]]"
status: Active
_width: wide
---

# ArtsAVL — Effort Estimation & Budget Gap

## Document Inventory

| # | Document / Source | Type | UI Subtype | Format | Quality |
|---|-------------------|------|------------|--------|---------|
| 1 | 2026 Creative Portal Expansion RFP (issued 2026-07-27; proposals due 2026-08-15; contact Katie Cornell) | Client RFP / requirements | — | PDF, 12pp (+ text extraction) | Complete for intent and constraints; **thin on current-state detail**. §4 integration list contains two verified GPS fleet-telematics vendors (Linxup, Rastrac) with no plausible function in an arts directory, indicating that section was compiled without verification. |
| 2 | Sol deal assessment, phases 1–5 (client profile, problem definition, current landscape, risk register, solution architecture) | Internal prior analysis | — | Markdown | Complete; gates G1–G5 passed |
| 3 | Solution architecture + architecture kickoff | Internal design | — | Markdown | Complete. Directed decisions: schema-per-tenant; all six goals delivered thin in six months |
| 4 | Site surface map — live crawl of `connect.artsavl.org`, 2026-08-03 | Route & functionality inventory | Screen List in Text (derived) | Markdown | **Complete for the public surface; member and admin surface confirmed to exist but interiors unobserved** (auth-gated, no login used) |
| 5 | Decision log (phases 1–5, incl. three corrections) | Internal | — | Markdown | Complete |
| 6 | Public inspection of `artsavl.org` (staff, board, membership, advertising, reports pages) | Client public web presence | — | HTML | Complete |

**User Story Inventory** — Present: **No** · Total: 0 · Format detected: N/A. The RFP states goals and desired outcomes, not stories. Functional requirements are derived from RFP goal statements, the observed route surface, and RFP §4's current-state description.

**Code origin:** no code was provided. The platform was built by a professional agency (Mindtonic Media, Asheville — Rails since 2007), so no-code/AI-platform origin markers do not apply and Step 3 AI-origin detection is not triggered. Code quality is asserted by the RFP (2,000+ RSpec examples, 154 end-to-end tests, maintained documentation) and **is unverified**.

---

## Materials Gap Analysis

### Present

- Client RFP with explicit goals, budget range, timeline, evaluation criteria and contractual terms
- Complete current-state technology stack as **stated** by the client (RFP §4)
- Independently verified framework currency: Ruby 4.0.5 (one patch behind 4.0.6, released 2026-07-14); Rails 8.1 series, **active support ends 2026-10-10** — inside the delivery window
- Full public route and functionality inventory from live crawl, including confirmed existence of the gated member and admin route surface
- Observed data-volume signals: profile IDs ≥ 2214, event IDs ≥ 25251
- Verified earned-revenue pricing: membership $75/yr single tier, nonprofit package $250/yr, web ads $100/month, newsletters 7,000+ subscribers at 60%+ open rate

### Missing — HIGH impact

- **Source code and repository documentation.** Provided to the selected firm at onboarding only (RFP §4). Every technical judgement — including the Reuse-over-rebuild recommendation and the entire tenancy work package — rests on the client's own description of its codebase. Forces the assessor to estimate the largest Epic against unverified premises.
- **All UI/UX materials.** No wireframes, mockups, prototype, design system or screen designs exist, on a project whose Goal 1 is an open-ended UX mandate that the RFP explicitly declines to specify ("recommend improvements based on user needs and product best practices rather than relying only on a predetermined feature list"). Gap rule applied: *hi-fi absent on a redesign project → HIGH*.
- **Member dashboard and admin panel interiors.** Confirmed to exist via auth-redirect behaviour, never seen. Critically, **RFP §4 states that reporting, automated/digest communications, and activity tracking exist, but no admin route for any of them surfaced during the crawl.** Goal 2 targets exactly those functions, so the delta between current and desired state — the thing being estimated — is unmeasurable from outside.
- **Business baselines.** No member count, membership revenue, renewal rate, advertising revenue, event/opportunity volumes, or traffic figures. Goal 3 (earned revenue) cannot be modelled and no success criterion can be quantified.
- **Licensing demand evidence.** No named partner organization, no target partner count, no willingness-to-pay signal. Goal 4 is the largest work package and its business premise is untested.

### Missing — MEDIUM impact

- WCAG conformance level unnamed and no accessibility audit exists, making remediation scope unbounded
- Newsletter/email marketing platform unidentified (7,000+ subscribers), so Goal 2 build-vs-integrate cannot be decided
- Donor system: a hosted form on `secure.lglforms.com` is observed and Little Green Light is the strongly-inferred vendor, but whether it is the CRM Goal 2 refers to is **not established**
- Consumers of the existing `/api/v1/events` endpoint unknown — any change is potentially breaking
- Outgoing partner's availability for a paid transition overlap unknown
- Purpose of the two GPS-telematics entries in §4 unresolved

### Assumptions Required

Each is flagged and carried into estimation rather than silently resolved:

1. **[A-01]** RFP §4's account of code quality (test counts, documentation, guarded pipeline) is accurate. *Basis: specific, verifiable-on-access claims, corroborated indirectly by observed platform behaviour — Hotwire/Turbo frames, ActiveStorage, Stripe hosted Checkout, current framework versions.* **If false, the tenancy and UX Epics move materially.**
2. **[A-02]** The account model decomposes into a tenant boundary as RFP §4's phrase "accounts serve as the ownership and billing root" implies. *Unverifiable without code.* Largest single estimation exposure.
3. **[A-03]** Reporting and communications exist in some admin form, per RFP §4, even though no route was found. Estimated as **enhancement of an existing surface**, not net-new build. If they turn out to be minimal, Goal 2 grows.
4. **[A-04]** Observed ID magnitudes indicate thousands of profile records and tens of thousands of event records; actual active counts are lower by an unknown factor.
5. **[A-05]** Partner organizations do **not** collect their own payments in version one (ArtsAVL remains merchant of record). Stripe Connect is excluded from this estimate and priced separately. *If the client requires per-partner collection in year one, scope is renegotiated pre-contract.*
6. **[A-06]** One partner organization is onboarded within the engagement, not several.
7. **[A-07]** ArtsAVL supplies a single named product owner at 8–10 hours per week sustained. *This is the assumption most likely to fail; it drives timeline, not hours.*

---

## Project Constraints

| # | Type | Description | Flexibility | Impact on Assessment |
|---|------|-------------|-------------|---------------------|
| C-01 | Hard deadline | Proposals due 2026-08-15. Work "substantially completed within six (6) months of contract execution" (RFP §3). Whether the six months is a funding-driven hard date is unconfirmed. | Soft (unconfirmed) | Estimate must fit a ~6-month calendar at 3.5 FTE; if the date proves hard, scope must flex rather than schedule |
| C-02 | Budget ceiling | $150,000–$250,000 all-in, covering platform assessment, discovery, planning, design, development, testing, project management and travel. Annual maintenance and support priced separately (RFP §11). | None (stated ceiling); §11 invites rescoping proposals | Total estimated cost must land inside the range at the chosen rate, or the proposal must explicitly rescope |
| C-03 | Technology mandate | Existing Ruby 4.0.5 / Rails 8.1 monolith retained; PostgreSQL; Hotwire over importmap with **no JavaScript build step**; Bulma; Stripe hosted Checkout; Heroku; GitLab; existing test suite and documentation preserved (RFP §4, §8). | None | No stack replacement is estimable; front-end options bounded by the no-build-step convention |
| C-04 | Team / client availability | ArtsAVL has approximately five staff, **none technical**, and no in-house product owner or IT function. Same two or three people must supply discovery, design review, taxonomy and moderation decisions, and acceptance testing. | None (structural) | **The binding constraint on calendar duration.** Sets client-side review latency; drives the Step 5 risk register and the timeline, not the hour count |
| C-05 | Infrastructure / data residency | No residency or on-premise constraint stated. Current hosting Heroku, media on DigitalOcean Spaces (NYC3), Redis via Upstash. | Soft | No compliance-driven infrastructure work estimated |
| C-06 | Development team | **1 TL + 2 developers + PM + designer.** Rails experience **Confirmed** (production systems shipped). | Fixed | **FTE = 3.5** delivery capacity used in all Step 6 timeline calculations. Confirmed stack familiarity permits full AI velocity calibration at CHECK 3 |
| C-07 | Intellectual property | RFP §12: ArtsAVL retains ownership of data, branding, content, accounts, existing codebase and **all enhancements**. Proposals must disclose any third-party software, licensing requirement, subscription cost or proprietary component affecting ArtsAVL's ownership, control or future use. | None | Prohibits proprietary components and encumbering dependencies; drove the decision to hand-roll tenancy rather than adopt a multi-tenancy gem |
| C-08 | Confidentiality | Confidentiality agreement required before repository access, credentials, or detailed operational information (RFP §12). | None | Blocks pre-contract code verification unless a finalist NDA is executed — the mechanism by which [A-01] and [A-02] could be retired before pricing |

---

## Business Analysis (Step 2)

**Business problem.** ArtsAVL's unrestricted earned revenue is capped by one county's willingness to pay $75/year, while its regional obligations grew after Hurricane Helene. It has simultaneously lost the only party with working knowledge of the platform that generates that revenue.

**Confirmed priority basis for this estimate:** all six RFP goals delivered at first-useful-version inside six months (directed 2026-08-03). ArtsAVL has published no ranking, and Goals 1 and 4 compete for the same budget — that competition is therefore distributed across the plan rather than resolved, and is the primary driver of the verdict below.

**Business priorities as stated by the client (RFP §5):** (1) functionality/usability/accessibility, (2) reporting/communications/engagement, (3) earned revenue, (4) licensable multi-organization platform, (5) content syndication, (6) ongoing maintenance.

**Unstated priority identified in prior analysis and carried as a finding:** directory completeness as evidence for advocacy and public appropriations. ArtsAVL is the *designated* county arts agency; a current directory partly justifies that designation. If this is the real north star, the roadmap reorders. Not estimated — flagged.

**Revenue-base observation bearing on priority.** Profile IDs ≥ 2214 at $75/year implies membership is plausibly a six-figure annual line [A-04]. Optimizing an existing base may return more, sooner, and at lower risk than building for unnamed partner councils. This argues against the equal-weight treatment being estimated here, and is recorded so the trade-off is visible rather than buried.

## Technical Analysis (Step 3)

**Stack (client-stated, RFP §4; framework currency independently verified):** Ruby 4.0.5 / Rails 8.1 monolith on Heroku; PostgreSQL; Puma; HAML + Bulma + Hotwire via importmap with **no JS build step**; Devise; Cloudflare Turnstile; Stripe hosted Checkout + webhooks; GoodJob on PostgreSQL; Upstash Redis; DigitalOcean Spaces via ActiveStorage; SendGrid; Sentry; Leaflet + Mapbox; GitLab.

**Verified currency:** Ruby 4.0.5 (2026-05-20) is one patch behind 4.0.6 (2026-07-14). **Rails 8.1 active support ends 2026-10-10 — inside the delivery window**, moving to security-only until 2027-10-10. The 8.2 upgrade is a dated, known event and is estimated in E-01/maintenance rather than deferred as a surprise.

**Feasibility verdict: technically feasible, low technology risk, high unknown-state risk.** Nothing in the evidence supports a rebuild; the platform is genuinely current. The risk is not the technology — it is that the largest work package (tenancy) is estimated against a codebase nobody outside the outgoing partner has read [A-01], [A-02].

**Non-functional targets confirmed for estimation:**

| NFR | Target | Basis |
|---|---|---|
| Accessibility | **WCAG 2.2 AA**, delivered as fixed-fee audit + prioritized plan, then remediation of the top-priority tier only; remainder a costed backlog | Confirmed 2026-08-03. RFP §7 requires "current WCAG standards" without naming a level (A15) |
| Availability | Maintain current posture; no formal SLA in this engagement | Non-goal; RFP §3 asks for a maintenance model, not an SLA |
| Performance | No regression against current baselines; listing pages must not regress when server-rendered | Derived; no client target stated |
| Tenant isolation | No cross-tenant data access on any code path; enforced by test assertions and fail-loud guards | Architecture §4.1 |
| Security | Preserve existing posture (SSL, encrypted credentials, bot protection, rate limiting, dependency scanning, static analysis); rotate/remove dormant credentials | RFP §4 |

## Scope Definition (Step 4)

**In scope** (confirmed 2026-08-03): all six goals at first-useful-version per architecture §9 — transition and continuity; platform assessment and bounded discovery; discoverability and structured data; schema-per-tenant foundation with data migration; per-partner branding, taxonomy and **one** partner onboarded; rollups and staff reporting; newsletter and donor-system integration; self-service advertising and membership revenue workflows; syndication feeds and one embeddable widget; accessibility audit plus top-tier remediation; maintenance and product-planning framework.

**Explicitly out of scope** (confirmed): Stripe Connect / per-partner payment collection [A-05] · partner self-service signup or admin portal · per-partner layout divergence · partner support desk · more than one partner onboarded [A-06] · cross-partner unified search · full visual redesign · accessibility remediation beyond the top tier · CRM or email-marketing platform built in-house · new membership pricing or tier design · bespoke per-outlet syndication integrations · bidirectional calendar ingestion · WordPress consolidation · ticketing, e-commerce, grants management, mobile app · 24/7 on-call or formal uptime SLA.

**TBD items with estimation impact:** TBD-01 partner payment collection model (A24) · TBD-02 target partner count year one (A8) · TBD-03 what varies per partner (A9) · TBD-04 newsletter platform identity (A17) · TBD-05 donor system and Goal 2 CRM meaning (A26) · TBD-06 current state of reporting/communications admin [A-03] · TBD-07 WCAG baseline (A15) · TBD-08 purpose of the two GPS-telematics integrations (D-05).

## Risk Adversarial Review (Step 5)

Basis: the 22-risk register from prior analysis, carried forward and re-scored against this estimate (confirmed 2026-08-03). **7 Critical, 7 High.** Full register in `../../assessment.md`.

**Re-scored for estimation impact — the four that move the number:**

| Risk | Estimation consequence |
|---|---|
| **B2** scope/budget mismatch | Now quantified below. This estimate confirms the risk empirically rather than by argument. |
| **T2** tenancy migration touches every table | E-04 is the largest Epic (304h adjusted) and rests on [A-02]. If the account model does not decompose cleanly, E-04 grows toward its pessimistic bound. |
| **T3** unverified codebase | Applies to the whole estimate. Drives the risk premium and the recommendation that later phases be ranged, not fixed. |
| **R2** client capacity, no product owner | Affects **calendar duration, not hours.** At 8–10 client hours/week the six-month window is achievable; below that, duration extends while hours stay flat. |

**Contingency requirements carried into Step 6:** buffer for T3 (unseen repo) · audit-then-price for T5 (accessibility) · spike-then-price gate for T2 (tenancy) · paid transition overlap or extended onboarding for R3.

## Effort Estimation (Step 6)

**Basis and honesty statement.** Hours are derived from `reference/estimation-reference.md` baselines, adjusted for project-specific signals: established stack familiarity (Confirmed) and a strong TL pull toward the low end of complex ranges; absence of any UI materials and an unverified codebase push toward the high end. **These are order-of-magnitude figures against a codebase that has not been read.** Assumptions [A-01] through [A-07] all sit underneath them.

`estimationScenario: AI_CALIBRATED` · `stackFamiliarity: Confirmed` (rates apply without modifier) · FTE 3.5.

### WBS Summary by Epic

| Epic | Goal(s) | Base hours | AI-adjusted | Notes |
|---|---|---|---|---|
| **E-01** Transition & continuity | 6 | 52 | **47** | Verified access across 11 services; live test transaction + test send; Ruby 4.0.6 patch; legacy domain retirement; **dormant-credential inventory and rotation** (D-05); runbook |
| **E-02** Assessment, discovery, audits, spike, validation | all | 280 | **258** | Codebase analysis across 8 domain modules (80h at the low end — good docs and tests asserted); written assessment; bounded UX discovery; 4 major flows wireframed; accessibility audit; **tenancy architecture spike**; 3 peer-council validation conversations; baseline capture; phased roadmap |
| **E-03** Discoverability & structured data | 1, 5 | 72 | **58** | Server-render first page of 3 indexes; per-tenant sitemap generation; schema.org on 3 content types; indexed-coverage measurement. Serves two goals with one body of work |
| **E-04** Tenancy foundation | 4 | 330 | **304** | Tenant registry; `search_path` middleware + job wrapper + fail-loud guards; public/tenant schema split; **data migration across 9 entity groups (80h)**; validation and reconciliation (28h); dry-run and cutover rehearsal (24h); rollback procedure (14h); cross-tenant isolation test suite (40h); per-schema CI migration step; key namespacing. **Largest Epic; gated on the E-02 spike** |
| **E-05** Per-partner config, branding, one partner live | 4 | 154 | **131** | CSS-custom-property branding with contrast validation; per-tenant taxonomy admin; per-tenant sender identity; domain resolution and custom-domain onboarding; tenant admin screens; one partner onboarded hands-on |
| **E-06** Rollups & reporting | 2 | 148 | **130** | Rollup tables + jobs; configurable staff dashboard; per-tenant scoping; membership/profile/event/opportunity/advertising analytics; CSV export |
| **E-07** Communications & integrations | 2 | 108 | **103** | Newsletter platform sync (40h — priced as a complex integration because the vendor is unidentified, TBD-04); donor↔member identity link; notification preferences; digest enhancement |
| **E-08** Revenue workflows | 3 | 136 | **116** | **Self-service advertising purchase (60h — new build, not polish: no member-side ad route exists)**; nonprofit package into checkout; renewal reminders and lapse recovery; advertiser performance view; conversion instrumentation |
| **E-09** UX improvements & accessibility remediation | 1 | 384 | **324** | Design 130h (10 screens, `designCompleteness: Wireframe` ×1.0 — nothing exists; +responsive; +design-system setup); frontend implementation of priority paths 150h; directory search/filter UX; mobile fixes; **top-tier accessibility remediation 60h**. Largest non-tenancy Epic, and the one with the least specification |
| **E-10** Syndication | 5 | 100 | **85** | Per-tenant iCal; JSON Feed; embeddable iframe widget; per-consumer keys with rate limiting and usage visibility; regression-protect the existing v1 endpoint |
| **E-11** QA & testing | all | 300 | **270** | ~18% of dev hours, plus E2E additions and UAT coordination. Includes extending the inherited 2,000-example suite with cross-tenant isolation assertions |
| **E-12** PM & overhead | all | 474 | **472** | PM at 15% (4+ contributors, active client); TL code review 144h; release management. **AI velocity = 0% by rule** |
| **Total** | | **2,538** | **2,298** | |

**Anti-overoptimism check:** adjusted total is 90.5% of base (floor 70%) and no single Epic is reduced by more than 30%. Within thresholds.

### Three-Point Estimate

| Scenario | Hours | Basis |
|---|---|---|
| Optimistic | **2,020** | Repo access granted pre-contract; [A-01]–[A-03] hold; client sustains 8–10 hrs/week; reporting/comms are genuinely enhancement-only |
| **Most likely** | **2,298** | The WBS as estimated |
| Pessimistic | **3,100** | Account model resists clean tenant decomposition; Goal 2 reporting is closer to net-new; UX scope expands under an unbounded mandate; client review latency extends rework |
| **PERT expected** *(O + 4M + P)/6* | **≈ 2,385** | |
| Management reserve (15%, justified by 7 Critical risks) | **+358** | Not reduced by AI calibration, per reference rules |
| **Total with reserve** | **≈ 2,743 hours** | |

### The finding that governs the verdict

**At any plausible blended rate, 2,743 hours does not fit $150,000–$250,000.**

| Blended rate | Implied cost of 2,743h | vs. $250K ceiling |
|---|---|---|
| $95/h | ~$261,000 | over |
| $125/h | ~$343,000 | 37% over |
| $150/h | ~$411,000 | 65% over |

Inverted: the budget ceiling of $250,000 buys roughly **1,650–2,000 hours** at $125–150/h. The all-six-thin scope needs **~2,750**. **The gap is roughly 700–1,100 hours, or 30–40% of scope.**

*(No Dualboot rate card was used or assumed — the table is rate sensitivity, and internal deal-value figures are excluded from this assessment by policy. Apply the real blended rate to convert.)*

This is risk **B2** moving from an argument to an arithmetic result. It was scored Critical before this estimate existed; the estimate confirms it independently.

### What does fit the budget

Deferring the two Goal-4 Epics (E-04 + E-05 = 435h) and syndication (E-10 = 85h) yields **~1,778h before reserve, ~2,045h with reserve** — approximately $256K at $125/h, still marginally over. Deferring E-10 and trimming E-09 to the accessibility audit plus the highest-value flows brings it inside $250K.

The version that fits, in shape: **continuity, assessment and discovery, discoverability, reporting, revenue workflows, accessibility audit plus top-tier remediation, and a tenancy architecture decision document** — with the tenancy build, partner onboarding and syndication sequenced into a defined, costed second phase. That is the honest form of the proposal §11 explicitly invites.

*Step 6 complete pending scenario confirmation. Step 7 (verdict) pending.*


---

## Correction — recomputed at $85/h (2026-08-03)

The rate sensitivity table above spans $95–150/h and concluded the scope was 37% over budget. **Dualboot's blended rate can be as low as $85/h, which reverses the conclusion.**

| Scope | Hours | Cost at $85/h | vs $250K ceiling |
|---|---|---|---|
| All six — most likely | 2,298 | $195,330 | inside |
| All six — PERT expected | 2,385 | $202,725 | inside |
| **All six — PERT + 15% reserve** | **2,743** | **$233,155** | **inside, ~7% headroom** |
| All six — pessimistic | 3,100 | $263,500 | **over by $13,500** |
| All six — pessimistic + reserve | 3,565 | $303,025 | well over |
| **Sequenced scope + reserve** | **2,045** | **$173,825** | inside, comfortable |

Inverted: **$250K buys 2,941 hours at $85/h**, not the 1,650–2,000 stated above at $125–150/h. All six needs 2,743. **It fits.**

**What was wrong and what wasn't.** The hour estimate is unchanged and was explicitly published as rate-agnostic ("no Dualboot rate card was used — apply the real blended rate to convert"). What was wrong was the *conclusion* drawn from it, which then became the most load-bearing sentence in [[artsavl-proposal]] — that the full programme could not be delivered at a quality worth signing. At $85/h that is not true, and the proposal has been rewritten.

**What survives the correction:**

- **The sequencing argument, which was never about money.** Building the tenancy model depends on three unresolved things: how the ownership model is actually structured (unverifiable without repo access — assumption [A-02], sitting under the largest Epic), whether partners collect their own payments ([A-05]), and whether peer councils will pay at all (risk B1). Building before those answers risks building twice. This holds at any rate.
- **Thin headroom at the ceiling.** $233K against $250K leaves ~$17K, and the management reserve is already inside that number. The pessimistic case breaches. Bidding all six at the ceiling means scope growth becomes a change order against an exhausted budget.
- **The budget floor now matters more than the ceiling.** ArtsAVL stated **$150,000–250,000**. At $85/h the floor buys **1,765 hours** — which does not cover all six (2,743h) but does cover the sequenced scope. If ArtsAVL is planning toward the low end, the sequenced shape is the only one that fits. Question A3 (does the upper end need additional board approval) is now the most commercially significant open question in the set.

**Net effect on the verdict:** still **CONDITIONAL GO**, but the condition changes. It is no longer "rescope to fit the budget." It is "confirm which end of the budget range is real, and gate the tenancy build behind repo access and the payment-model decision."
