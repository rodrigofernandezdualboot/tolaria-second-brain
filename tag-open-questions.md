---
type: Note
related_to: "[[tag]]"
status: Open
_organized: true
---

# TAG — Open Questions

Running log of open questions for the TAG Assessment Core Modernization RFP. Each stays named as open until answered, with an owner and a way to resolve it. Feeds the Open Questions section of the Technical Intake Brief.

## OQ-01 — Does Phase 1B Test Delivery scope include all Entry Level questionnaires?

**Status:** Open · raised 2026-07-03

**Question:** For the Entry Level slice (Phase 1B), which candidate questionnaires/instruments must the rebuilt Test Delivery system support — and are all of the following in scope or only a subset?
- Production Situational Judgment Questionnaire (C)
- Production Personality Questionnaire (C) — `test_id=117`
- Production MTM (C) — `test_id=119`
- Object Comparison
- Perceptual Speed & Accuracy — `test_id=4`

**Why it matters:** Test Delivery is the largest 1B workstream, and each in-scope battery needs golden test cases for the bit-identical parity gate. The instrument list directly sizes both the build and the parity effort, so an unbounded "all questionnaires" reading materially changes the estimate.

**What we know from the provided docs:**
- No source document names these instruments or gives an explicit 1B inclusion list.
- Scope is defined by **job/band type** (7 types: Clerical, Detailer, ELA3, VVG2/VVG3 Rigger variants), not by named test. Instruments are grouped into **batteries** via `assessments` / `assessment_tests` (process T-22, Admin config).
- **PQ16** (TAG's proprietary personality questionnaire) is internally scored at finalization via TAGScoringService — almost certainly in scope. `[ASSUMPTION]` the "Personality Questionnaire (C)" above is PQ16, not 16PF.
- **16PF** (external, via NetAssess/Talogy, process T-04/IPAT) is explicitly **deferred** — not Entry Level Phase 1. Any instrument scored through 16PF is out of 1B.

**Owner / how to resolve:**
- Add to the questions-to-submit list for TAG.
- Confirm the definitive instrument → battery → band-type → in/out mapping with Bernie in the Phase 1A source-of-truth session.
- Map the `test_id` values (4, 117, 119) against `assessment_tests` once the sandbox/schema is available under NDA.

Related: [[tag]]

## OQ-02 — Honorlock integration: credentials, browser dependency, and embedding

**Status:** Open · raised 2026-07-06

**Question:** What are the confirmed constraints for integrating Honorlock into the rebuilt Test Delivery app, specifically: (a) how/when do we obtain API credentials, (b) is the candidate-side Chrome extension acceptable to TAG and the primary client, and (c) what are the iframe/CSP/domain-allowlist and SDK-distribution requirements?

**Why it matters:** Honorlock is the catalyst for the whole RFP. The Phase 1 requirement is a vendor-agnostic proctoring integration point, stub-verified at acceptance (live connection in Phase 3). These constraints govern how far we can build and validate the adapter before a Honorlock contract exists.

**What we know (from public docs, de-risk research 2026-07-06):**
- Honorlock supports a **custom/standalone path — no LMS or LTI required**: a public **REST API + "Elements" JS library + Exam-Taker SDK + signed webhooks** (`docs.honorlock.com`). Auth is OAuth2 client-credentials; platform IDs map via `external_id`; results via 13 webhook events or a Sessions API. This maps cleanly to a vendor-agnostic `IProctoringProvider` adapter.
- Confirms our Phase 1 design: **build the adapter + webhook receiver, stub the live connection** (T-24), connect the real vendor in Phase 3.

**Residual unknowns to confirm:**
- **API credentials (`client_id`/`secret`) are almost certainly gated behind a signed contract** — full live validation likely waits for Phase 3 / contract.
- **Mandatory candidate-side Chrome extension** (webcam/mic; Chromium constraint) — UX and support implications that intersect the Test Delivery rebuild and the design refresh.
- **Iframe/CSP embedding + Elements/SDK distribution** (npm vs CDN, `frame-ancestors`, domain allowlist) not published — confirm during 1A design.

**Owner / how to resolve:** confirm with Honorlock (via TAG's committed relationship) and validate embedding/CSP details in Phase 1A architecture design. Sources: honorlock.com/custom-integrations, docs.honorlock.com (enablement / exam-taker / webhooks / api).

Related: [[tag]]

## Assumptions register

Tagged `[ASSUMPTION]` items from the Phase 1 scope boundary (see `TAG-Technical-Intake-Brief.md`). Each must be confirmed or it changes scope, cost, or timeline.

| # | Assumption | Impact if wrong | Owner | Resolve by |
|---|---|---|---|---|
| A1 | WCF `TAGScoringService` wraps cleanly (rehostable, not rebuild) | +4–8 backend weeks; median → conservative (~$189K) | Vendor | 1A source review (re-baseline checkpoint) |
| A2 | Scoring module already isolated with buildable parity/unit tests | Parity effort grows; golden-case approach shifts | Vendor + Bernie | 1A KT + source review |
| A3 | "Entry Level" = the 7 documented band types; instrument subset TBD | Test Delivery + parity scope changes | TAG + Bernie | 1A (OQ-01) |
| A4 | "Personality Questionnaire (C)" = internal PQ16, not 16PF | If 16PF, it's deferred (out of 1B) | TAG + Bernie | 1A (OQ-01) |
| A5 | Single shared Azure SQL + ABP TenantId isolation acceptable for P1 | If DB-per-tenant required, more infra + ABP Commercial | TAG / Leadership | Proposal / 1A |
| A6 | ABP open-source (shared-DB tenancy) sufficient; Commercial SaaS module not needed in P1 | Licensing cost + disclosure change | Leadership + Vendor | Proposal / 1A |
| A7 | TAG provides cloud-to-legacy connectivity (VPN / whitelist / replication) | Parallel run blocked | TAG | 1A |
| A8 | TAG completes pre-conditions before kickoff (Logic App migration, credential remediation) | 1A start slips | TAG | Pre-kickoff |
| A9 | Bernie available for committed KT hours in 1A | Parity confidence + fixed-fee realism drop | TAG | Proposal (your Q6) |
| A10 | Source code + sandbox provided post-award under NDA; fee re-baselined after 1A | Estimate is provisional | TAG / Vendor | Award → 1A |
| A11 | Only band score (1–6) returns to SF; no PDF/doc assets | Report engine would enter scope (XL) | TAG | Confirmed Q&A #3 (monitor) |
| A12 | Stubbed proctoring adapter acceptable at P1 acceptance; live Honorlock in Phase 3 | Phase 3 pulled forward; credentials needed early | TAG + Honorlock | 1A (OQ-02) |
| A13 | Staffing includes a 2nd backend dev; 14-week schedule | 1B slips ~4–5 weeks | Leadership | Proposal decision |
| A14 | Spanish-language Entry Level path deferred (out) | Adds bilingual build + QA | TAG | 1A |
| A15 | Light UI/UX refresh is an optional value-add, not baseline | Effort/price change if made baseline | TAG / Leadership | Proposal |
| A16 | Admin functionality limited to Entry Level config needs | Broader admin rebuild expands scope | TAG + Bernie | 1A |

Related: [[tag]]
