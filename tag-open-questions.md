---
type: Note
related_to: "[[tag]]"
status: Open
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
