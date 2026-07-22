---
type: Note
status: Active
relationships:
  belongs_to:
    - "[[biopredictx-alpha-2-0]]"
  related_to:
    - "[[biopredictx-alpha2-phased-proposal]]"
    - "[[biopredictx-alpha2-requirements-estimation]]"
    - "[[biopredictx-fdc-alpha-2-0]]"
_organized: true
---
# BioPredictX Alpha 2.0 — Risk Register

Consolidated list of the risks discussed across the Alpha 2.0 notes \([[biopredictx-fdc-alpha-2-0]], [[biopredictx-alpha2-requirements-estimation]], [[biopredictx-alpha2-phased-proposal]], [[biopredictx-scope-change-client-summary]]\). Phase 1 exists largely to retire R1–R6 before the bigger build commits.

## Phase 1 risks \(validated by the feasibility simulation\)

| \# | Risk | Mitigation / validation |
| --- | --- | --- |
| R1 | **Therapy timing \(NFR-A2\)** — COTS/Garmin sleep staging may not detect emerging deep sleep early enough to fire therapy *before* onset. The FDC itself flags this: the assumption that "on"-signal logic can be derived from simple alterations to standard COTS staging may fail, forcing custom algorithm development in this phase. | Phase 1 Pi simulation measures detection latency against expert-scored ground truth. Contingency: relax NFR-A2 to "at onset" with client sign-off. |
| R2 | **Algorithm expertise gap** — we don't have in-house sleep-science expertise. Whatever algorithm we select needs someone on the client's side with that knowledge to validate it is clinically viable for the product. | Name a client-side clinical/algorithm reviewer up front; make their sign-off part of the Phase 1 gate review \(validated algorithm spec\). |
| R3 | **Garmin signal availability** — we may not be able to get the parameters/signals we need from the Garmin devices. Connect IQ capture is foreground-only with limited sensor access. | Phase 1 Connect IQ sensor-sampling spike \(FR-A12 feasibility\). |
| R4 | **Garmin app must stay in the foreground overnight** — the capture app running foreground all night \(likely with backlight/CPU active\) may drain the watch battery before the session ends. | Validate in the Phase 1/2 spikes; Phase 2 overnight streaming trials explicitly measure cadence, battery, and latency. |
| R5 | **Watch→Pi real-time transport \(FR-A13\)** — Garmin may not be able to stream in real time; BLE \(Pi-as-peripheral\) and Wi‑Fi HTTP both look viable per derisking, but neither is proven on the bench. | Bench transport spike in Phase 1; lock the approach before Phase 2 builds on it. |
| R6 | **Wearable selection may change** — Garmin lacks the EEG needed for continued deep-sleep testing; Muse EEG has all sensors in one device. If Garmin can't deliver \(R3/R5\), the switch to Muse reshapes the Phase 2 capture work. | Phase 1 runs both Garmin and Muse dev environments and locks the wearable decision at the gate review. |
| R7 | **Ground-truth data availability** — the algorithm validation depends on recorded sleep datasets with expert-scored sleep stages to replay and score against. | Confirm dataset sourcing \(and who does expert scoring\) in Wk 1 of Phase 1; ties back to R2. |

## Later-phase and project-level risks

| \# | Risk | Why it matters | Mitigation |
| --- | --- | --- | --- |
| R8 | **COTS-first tension \(NFR-A3\)** — the COTS-first objective is in direct tension with the timing constraint \(NFR-A2\) and with FR-A12/A13: the Garmin capture app is itself custom software the FDC didn't anticipate. Risk of custom-software creep. | Erodes the low-cost premise of the constrained scope. | Phase 1 settles the COTS-vs-custom decision explicitly; scope changes go through the client. |
| R9 | **SE-added scope \(FR-A12/A13\)** — the capture app and transport are not in the FDC; they were surfaced during derisking. Client expectations and budget must include them \(~96 avg person-days combined\). | Largest single estimate items with Medium risk and wide Low–High spreads. | Already called out in [[biopredictx-scope-change-client-summary]]; keep them visible as separate line items. |
| R10 | **Safety / fault handling \(NFR-A7, FR-A11\)** — therapy must shut down safely from a fault in any state. Bounded but safety-relevant work on a real device driving magnets on a sleeping person. | A fault path that misbehaves is a study-stopping \(and safety\) event. | Dedicated fault-path testing in Phase 3 \(Wk 5 and Wk 7\); administrative controls \(NFR-A4\) reduce the engineered surface. |
| R11 | **Beta algorithm seam \(NFR-A9\)** — the interface point for the future proprietary "black box" algorithm must be defined now, without the algorithm existing; the seam may guess wrong. | A wrong interface forces rework at the Beta transition. | Keep the analytics interface thin and data-shaped \(timestamped stages in/out\); revisit at each phase review. |
| R12 | **Overnight session reliability** — beyond battery \(R4\), the live path must sustain the required cadence and latency for a full night, with buffering covering dropouts. | Sessions repeat the sleep-stage loop 3–6× per night; gaps corrupt the study data. | Phase 2 success criteria require data sustained overnight within battery/latency limits; ingestion buffering \(FR-A6\). |

## Notes

- R1, R3, R5, and R6 are interlocked: the Phase 1 exit gate \(feasibility report + gate review\) is the single decision point that resolves them together — timing verdict, wearable locked, transport proven, algorithm spec validated.
- Risks from the old production scope \(thundering-herd ingestion, de-PII/crypto-shredding, GDPR+FDA dual-track, ML scope creep\) belong to [[biopredictx-scope-v2]] and do not apply to Alpha 2.0.
