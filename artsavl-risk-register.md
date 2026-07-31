---
type: Note
related_to: "[[artsavl]]"
status: Active
---

# ArtsAVL — Risk Register

Qualitative likelihood × impact scoring, 2026-07-31. Every risk traces to a finding in [[artsavl-technical-assessment]]. Architectural responses in [[artsavl-solution-architecture]].

**22 risks across four categories. Seven score Critical — and that is itself the finding.** A $150–250K, six-month engagement should not carry seven criticals. The reason it does is that the RFP describes enterprise-shaped work for an organization with five non-technical staff and no product owner. Every one is manageable; none is manageable by pretending the scope fits.

## Critical

| ID | Risk | Mitigation |
|---|---|---|
| **B2** | **Scope and budget mismatched.** Six goals including a tenancy re-architecture and an open-ended UX mandate, six months, $150–250K. A competitor promising all six may win the paper comparison. | §11 of the RFP explicitly invites a rescoped proposal. Decision taken: bid all six thin, with written in/out boundaries per goal as the control. **Residual: High, accepted by decision.** |
| **T1** | **Per-partner payment collection is unaddressed anywhere in the RFP.** §2 says partners will earn revenue "through their own memberships and advertising" — money reaching partner bank accounts through shared software means Stripe Connect or per-tenant accounts, with onboarding, KYC, payouts, refunds, disputes and tax reporting per partner. | v1 keeps ArtsAVL as merchant of record, remitting to partners. Tenant record carries payment configuration so Connect is additive later. Named in the proposal as a priced option. |
| **T2** | **Tenancy migration touches every table.** The boundary lands on the account model, already "the ownership and billing root." One missed scope is a cross-tenant data leak. | Conditioned on a paid architecture spike with repository access, delivering a tenancy decision document before build. Cross-tenant leakage tests added to the inherited suite. |
| **F1** | Six unranked goals; Goal 1 and Goal 4 compete directly for the same fixed budget. | Written inclusion/exclusion lines per goal. Covering both distributes the competition rather than removing it. |
| **F2** | Goal 1's UX scope undefined by design — no feature list, no user research. | Bounded discovery, fixed fee, defined participants, design decision gate. Also the direct answer to the 15% discovery-approach criterion. |
| **R1** | **No internal technical champion; the economic buyer is also the technical evaluator.** An unchallenged recommendation is an unvalidated one, so scope disputes surface as change orders instead of decisions. | Every artifact written for a non-technical reader who must defend it alone. Written decision record at every gate. Structural — cannot be removed. |
| **R2** | **~5 non-technical staff must supply the client-side half** of a six-month engagement touching everything. | Required client roles and hours named in the proposal as a stated assumption, with what slips if unmet. Product owner at 8–10 hrs/week sustained is the assumption most likely to break. |

## High

| ID | Risk | Mitigation |
|---|---|---|
| **T3** | The entire assessment rests on the RFP's description of its own code quality. Nothing has been seen. | Request finalist repo read access under NDA. If refused, assessment becomes a discrete first phase with a re-baselining decision point. |
| **T4** | Credential and account handoff across eleven external services with silent failure modes — a Stripe webhook or SendGrid DNS misconfiguration stops revenue or member email with no visible error. | Verified transition checklist as the first contractual deliverable, with a live test transaction and test send proving each path. |
| **T5** | Accessibility conformance level unnamed, no audit baseline, UI uninspected. Remediation on an existing product is effectively unbounded. | Audit plus prioritized plan fixed-fee; remediation priced after. **Note: an accessibility audit via Liam's contact is already contemplated in [[artsavl-overview]] — that is this mitigation, already sourced.** |
| **F3** | Non-goals unaccepted — most expensively, that we do not become the partner support desk. | Non-goals stated in the proposal so acceptance is contractual. Partner DNS onboarding named as a support interaction someone must own. |
| **R3** | The outgoing partner's knowledge is decaying from its peak right now; availability unknown. | Paid transition overlap inside onboarding, or an extended onboarding phase priced honestly. |
| **B1** | **The licensing thesis — the north star — may be unvalidated.** No named partner, no published target, and peer councils as budget-constrained as ArtsAVL. | Three structured peer-council conversations on requirements and willingness to pay, in the first six weeks. Katie Cornell convenes these organizations, so access is not the obstacle. |
| **B3** | May not be a live competitive process. Cold RFP, unexplained incumbent exit, and a question deadline that coincides with the proposal deadline. | Cap pre-award investment until the Q&A response gives a signal. **Materially reduced by the Aug 11 meeting recorded in [[artsavl-overview]] — see [[artsavl-assessment-findings-vs-initial-read]].** |

## Medium and below

Directory discoverability remediation scope (T6) · Linkup and Rastrac unknown blast radius (T7) · Goal 2 build-vs-integrate split and unidentified newsletter platform (F4) · syndication may address supply when the constraint is partner demand (F5) · member content rights for syndication (F6) · presumed operational champion unconfirmed (R4) · incumbent designer politics (R5) · flat licence fee against metered per-tenant infrastructure (B4) · maintenance agreement priced without support history for a finance-literate board (B5) · our Q&A answers reaching competitors (B6).

## Contingency notes

Estimation must consume these rather than estimate around them.

- **Accepted:** B3 with capped pre-award exposure · B6 (question answers reaching competitors) · R5 (designer politics, handled by collaboration).
- **Buffer required:** T3 — price the assessment phase firmly, later phases as ranges with a re-baselining gate. T5 — audit fixed, remediation priced after; no accessibility remediation inside a fixed price. T2 — architecture spike fixed, tenancy build priced after; this decision gate is non-negotiable. R3 — either a paid overlap with the outgoing partner or an extended onboarding, both of which cost money.
- **Baseline capture:** the "staff time reduction" success criterion is unverifiable without a discovery baseline. Budget the baselining or drop the criterion; do not assert it.
