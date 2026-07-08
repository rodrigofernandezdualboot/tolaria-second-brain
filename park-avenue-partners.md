---
type: Project
---
# Park Avenue Partners

> **Structure:** shared Platform Foundations + two scoped modules \(Billing Engine, Leak Detection Engine\). The two modules are separable for phasing and pricing; the foundations are built once and serve both.**Consumer:** the BMad analyst starting a product brief. Every section is written to be actionable without the discovery call in the room. **Reconciliation note:** this brief merges two sources that disagreed — the Estimation Brief \(`ParkAvenuePartners_EstimationBrief.docx`\) and the discovery meeting note \(`net-new-estimate`\). Where they conflicted, the resolved value and its source are stated inline. Inferences are tagged `[ASSUMPTION]`.

## Deal Context

Net-new client, no prior relationship with Dual Boot. Park Avenue Partners operates ~24 mobile-home-park communities \(~2,400 pads total, ~100 submeters per park\) and expects the portfolio to grow through acquisition. They have no existing AI infrastructure and no enterprise relationships with cloud providers or model makers — this is a greenfield build on an AWS account they will create.

**Commercial frame:** budget is under $100K, ideally under $50K \(source: meeting note; the Estimation Brief omits budget entirely\). The client wants a low-maintenance, turnkey system. This ceiling is the dominant design constraint and is the reason the two modules are kept separable — it may force phasing rather than a single fixed-fee build.

**People / roles:** Owner and VP of Operations are the primary alert recipients and decision-makers; Regional Managers and Community Managers receive property-specific leak alerts. On the Dual Boot side, Peter Klayman is driving; Rodrigo owns technical validation, architecture, and proposed approach \(not the final number for now\); a separate estimator assigned by Maddie runs in parallel for the first ~6 weeks. A joint call \(Rodrigo, Peter, estimator\) is targeted for Wed/Thu 8–9 July.

**Why now:** the client is losing money today. Utility rate changes go undetected, causing months-to-years of tenant underbilling; distribution leaks in owner-owned pipes are billed to the owner with no automated way to catch them. Clear, quantifiable ROI is the buying trigger.

## Business Problem

Park Avenue is running a portfolio-scale metering-and-billing operation on manual process. Two jobs to be done:

1. **Stop revenue leakage from stale rates.** Communities are master-metered; the owner receives one utility bill per property and re-bills tenants using per-gallon rates and fixed assessments entered by hand into Rent Manager. When a utility changes its rate or adds a fee, nothing flags it, so tenants are underbilled until someone happens to notice. The job: detect rate/fee changes monthly and keep tenant billing current automatically.
2. **Stop paying for water that leaks.** Three metering tiers \(city master → owner master → pad submeters\) are never cross-referenced, so distribution leaks, city-meter faults, and anomalous tenant usage go undetected. The job: reconcile the tiers monthly/daily and alert the right people early, while the fix is still cheap.

If this works, the owner recovers underbilled revenue, catches leaks before they compound, and reduces surprise-billing disputes with tenants — without adding headcount.

**Framing flag:** the client uses "AI" language, but this is a deterministic data-processing project. The CTO review confirmed it as an ML/data pipeline, not an agentic system. OCR/AI applies narrowly \(PDF bill parsing\). The proposal and SOW should reflect this to set accurate expectations and avoid overselling.

## Current Technical Landscape

**Tenant / billing platform: Rent Manager**. All integration below assumes Rent Manager.

**Metering — three tiers:**

- **Tier 1 — City master meter.** Read from the monthly utility bill, in arrears.
- **Tier 2 — Owner-owned master meter** immediately downstream, near-real-time reads.
- **Tier 3 — Pad-level submeters** from three systems: **Metron/WaterScope, NES, and Dune**. Metron and Dune provide daily reads. All three confirmed to have API or dashboard access. WaterScope \(Metron\) already has some native leak alerting.

**Utility billing delivery — mixed.** Bill formats vary by municipality \(HTML and/or PDF\) regardless of delivery channel. **~50/50 — roughly 12 portal, 12 email-only** 

**Data flows today:** utility → owner \(bill, monthly\) → manual rate entry into Rent Manager → tenant charges. Master and submeter data sits in three vendor systems, uncorrelated. No automated pipeline exists.

**Verified vs. described:** meter-system and Rent Manager API *availability* is asserted in the source brief but not independently verified by us. Portal/email split, pad counts, and budget come from discovery, not from system inspection. 

## Where the Solution Fits

A headless, scheduled ML pipeline on AWS that sits between the client's data sources \(utility bills, three meter systems\) and Rent Manager. It reads from the sources, applies deterministic threshold/pattern logic, and writes results back into Rent Manager \(rates, charges, tenant notifications\) plus email alerts. No custom UI — output surfaces through Rent Manager and email, which fits the "turnkey, low-maintenance" requirement and keeps cost down.

The logic is deterministic and auditable, which lowers risk, cost, and maintenance — the right profile for the budget and for a client with no AI infrastructure.

**Integration surface:**

- **Ingestion in:** 3 meter systems \(separate handler per system\), ~12 utility portals \(scrape\), ~12 utility inboxes \(email parse\).
- **Compute:** AWS pipeline, Bedrock for the narrow model/OCR needs, cost-capped per run.
- **Write out:** Rent Manager REST API \(rates, fixed-charge line items, tenant email/SMS notifications\).
- **Alerts out:** email to role-based recipient lists, per property.

## Scope and Assumptions

### Platform Foundations \(shared — built once, serves both modules\)

**In scope:**

- AWS account configuration \(client creates account; Dual Boot leads setup\), Bedrock access, consolidated billing.
- Secrets management \(AWS Secrets Manager\) for utility portal credentials, scoped read-only where possible.
- Per-run cost caps / hard budget thresholds to prevent runaway spend.
- Meter-system ingestion handlers — one per system \(Metron/WaterScope, NES, Dune\).
- Rent Manager REST API integration layer \(read structure; write rates, charges, notifications\).
- Monitoring via AWS CloudWatch \(runtime, error states, cost\).
- **Property onboarding workflow** — repeatable, documented: add utility portal/email credentials, configure meter API, create Rent Manager community entry, initialize rate baseline. First-class requirement, not an afterthought \(portfolio is growing by acquisition\).

**Out of scope:**

- Custom end-user UI / dashboard \(system is headless by design\).
- Autonomous/agentic decision-making \(deterministic logic only\).
- Ongoing portal-maintenance and model-governance labor \(candidate for a separate T&M/retainer line — see Risks\).

### Module A — Billing Engine \(UC1\)

**In scope:** monthly retrieval of each utility bill via **two ingestion paths** — portal scrape \(~12\) and email/attachment parse \(~12\) — feeding one shared bill-parsing layer \(HTML + PDF, OCR where needed\); detect rate and fee/assessment changes against stored baselines; apportion fixed/improvement fees across units; apply current per-gallon rate; write updated rates and fixed-charge line items to Rent Manager; alert Owner/VP on detected rate changes.

**Out of scope:** reconciling historical underbilling retroactively \(unless explicitly added\); handling utilities that neither email nor expose a portal \(none identified — flag if found\).

### Module B — Leak Detection Engine \(UC2\)

**In scope, three layers:**

- **Layer 1 — City vs. owner master:** compare city bill \(monthly, arrears\) to owner master reads; sustained discrepancy → possible city-meter fault/billing error → alert Owner, VP.
- **Layer 2 — Owner master vs. sum of pad submeters:** delta indicates a distribution leak in owner-owned pipes; nighttime-window analysis \(e.g., 2–4 AM\) for high-confidence signal → alert Owner, VP, Regional Manager, Community Manager for that property.
- **Layer 3 — Pad usage anomaly:** each pad vs. its rolling ~30-day baseline; significant spike \(e.g., ≥2× average\) → courtesy tenant notification via Rent Manager \(email/SMS\). Daily reads from Dune/Metron enable near-real-time detection.

**Out of scope / open:** whether Layer 3 **supplements or replaces** WaterScope's native alerting \(determines Layer 3 build scope — see Open Questions\).

### Assumptions \(load-bearing — each one breaks the estimate or design if wrong\)

- `[ASSUMPTION]` Rent Manager REST API \(rmAPI\) supports **write** of rate line items and fixed charges, **and** tenant SMS/email notifications, at the needed granularity. Rent Manager exposes both legacy SOAP and a newer REST API with documented read/write; which endpoints cover our three needs \(write rates, write notifications, read community/pad structure\) is unverified. *This is the single most load-bearing assumption.*
- `[ASSUMPTION]` All three meter systems expose a usable API \(not dashboard-only\) for automated daily/near-real-time pulls; separate handler per system is sufficient.
- `[ASSUMPTION]` Utility portals allow read-only credential scoping and permit automated login without MFA that blocks headless access.
- `[ASSUMPTION]` The ~50/50 email/portal split holds and every utility uses at least one of the two channels with retrievable bill history.
- `[ASSUMPTION]` Bill-format variance across municipalities is bounded enough that a finite set of parsers \(built after a format audit\) covers the portfolio.
- `[ASSUMPTION]` ~100 submeters/park and ~2,400 pads total is representative for sizing compute and parsing effort.

## Proposed Solution Shape

**Architecture \(high level\):** scheduled AWS pipeline, deterministic. Ingestion layer \(3 meter handlers + portal-scrape + email-parse\) → shared parsing/normalization \(Bedrock for OCR/parse assist only\) → detection/comparison logic \(rate-change diff for A; three-layer threshold/baseline comparison for B\) → write-back \(Rent Manager\) + alerting \(email, role-based\). Secrets Manager for credentials, CloudWatch for observability, per-run cost caps.

**Delivery model — options, with the budget in mind:**

- **Option 1 — Both modules, single fixed-fee build.** Cleanest for the client; highest risk of colliding with the sub-$100K ceiling given the dual-ingestion UC1 surface.
- **Option 2 — Phase by ROI \(recommended to consider\).** Ship Leak Detection Layer 2 \(owner-master vs. submeters — the biggest direct dollar leak\) or the Billing Engine first, whichever the client values most, then add the rest. Fits the budget and proves value early. Trade-off: two mobilizations, slightly higher total.
- **Option 3 — Foundations + Module A first, Module B second.** Billing recovers underbilled revenue fast and reuses the same Rent Manager write layer B needs. Trade-off: leak savings deferred.

Recommend presenting Option 2/3 framing rather than forcing one — the estimator's numbers against the ceiling should decide.

**Team composition** `[ASSUMPTION]`**:** cloud/data engineer \(AWS pipeline, ingestion\), integration engineer \(Rent Manager + meter APIs\), part-time ML/data specialist \(parsing, anomaly baselines\), TL/architect oversight. Right-sized for a deterministic pipeline, not an ML research team.

## Risks and Dependencies

- **Bill-format variance \(High\) — primary UC1 scope risk.** Formats differ by municipality across *both* channels \(portal HTML and emailed PDF\). Retire with a **bill-format audit during build**, before finalizing the parsing layer.
- **Portal login reliability \(Medium, ongoing\).** Portals change session/structure; scrapers break. ~12 portals to maintain. Flag as a **T&M support line**, not fixed-fee.
- **Rent Manager API coverage \(High until verified\).** If REST can't write rates/notifications as needed, the core write path changes. Retire by confirming against rmAPI docs / a sandbox before the estimate hardens.
- **Credentials security \(Medium, accepted\).** Pipeline needs read-level access to portals tied to auto-pay accounts. Scope read-only, document exposure. Client acknowledged and accepted the tradeoff.
- **WaterScope overlap \(Medium\).** Native Metron alerting overlaps Layer 3 — supplement vs. replace determines build scope. Resolve before scoping Layer 3.
- **Property-acquisition onboarding \(Medium, recurring\).** Not one-time — every acquisition needs portal/meter/Rent Manager setup. Define as a **documented T&M line item** separate from the fixed fee to avoid scope disputes.
- **Model drift / pipeline monitoring \(Low-Medium, ongoing\).** Anomaly baselines and pipelines need periodic review. Offer a **light managed-service or T&M retainer** for post-launch governance.
- **Budget ceiling \(High, commercial\).** Sub-$100K / ideally sub-$50K against a dual-ingestion, three-layer, multi-system scope. Phasing is the main lever — see Solution Shape.

## Open Questions

- **Rent Manager REST API coverage** — does it support write of rate line items, fixed charges, and tenant SMS/email? *Owner: Rodrigo, before estimate hardens.*
- **Exact email/portal split and per-utility bill formats** — confirm the ~12/12 assumption and gather sample bills for the audit. *Owner: client / Peter.*
- **Layer 3 vs. WaterScope** — supplement or replace native Metron alerting? *Owner: client \(Ops\).*
- **Meter APIs vs. dashboard-only** — is each of Metron/NES/Dune truly API-accessible for automated pulls? *Owner: Rodrigo \(vendor-doc research\).*
- **Budget vs. scope** — one build or phased? Which module has priority ROI for the client? *Owner: Peter / estimator / client.*
- **Fixed vs. T&M boundary** — which items \(portal maintenance, onboarding, monitoring\) sit outside the fixed fee? *Owner: Peter / estimator.*
- **Notification channels** — is SMS in scope for tenant alerts, or email only \(affects Rent Manager notification path\)? *Owner: client.*

## Technical Champion

Not yet identified on the client side — likely the **VP of Operations** \(recipient of most alerts, owns the operational pain\). *[ASSUMPTION]* To equip them: their internal narrative is "this pays for itself by recovering underbilled revenue and catching leaks we currently eat," and the objection they'll face is "why not just have staff check portals manually" \(answer: 24 portals × monthly × changing rates doesn't scale, and it's already failing\). Confirm the champion on the Wed/Thu call.

## Evidence Status

None delivered. No demo/PoC planned yet. The highest-value early evidence would be a **bill-parsing spike** on a small sample across both channels — it retires the top scope risk \(format variance\) and the Rent Manager write assumption in one artifact. Consider proposing it as a paid discovery/build-phase step.

## Handoff Notes for Planning

Pick up first: **\(1\)** confirm Rent Manager REST API coverage — it gates the whole write path; **\(2\)** get sample bills and lock the ~12/12 split for the format audit; **\(3\)** decide phasing against the budget ceiling. Do not lose downstream: this is a **deterministic ML/data pipeline, not agentic** — PRD and architecture language must reflect that. Onboarding, portal maintenance, and monitoring are **recurring**, not one-time, and should be modeled as T&M outside the fixed fee. The two modules share one foundation but are separable — keep that seam intact so planning can phase if the budget requires.
