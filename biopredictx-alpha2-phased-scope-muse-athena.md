---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-muse-athena-eeg-pivot]]"
  - "[[biopredictx-alpha2-phased-proposal]]"
  - "[[biopredictx-alpha2-risks]]"
  - "[[biopredictx-alpha2-sleep-algorithm-selection]]"
  - "[[biopredictx-alpha2-huggingface-model-approach]]"
  - "[[biopredictx-fdc-alpha-2-0]]"
status: Draft
---

# BioPredictX Alpha 2.0 — Phased Scope v2 (Muse Athena EEG-First)

Revised phased implementation scope reflecting the client's hardware/architecture pivot from a **Garmin-wrist** wearable to a **Muse S Athena EEG-first** closed loop. Supersedes the Garmin-first plan in [[biopredictx-alpha2-phased-proposal]]; direction is taken from [[biopredictx-alpha2-muse-athena-eeg-pivot]].

> **Status caveat (per Lindsey Wilkinson, 21 Jul 2026):** the client's three handoff documents are agent-generated captured context, **not** locked requirements pending her review and internal discussion. This scope is therefore a **draft for alignment**, and several items below are explicitly gated on client/clinician sign-off. Firm hours and cost are **deliberately withheld until after the Phase 0 spike** — Athena specs are engineering-verification items and the therapy-control logic is "TBD — clinician input required."

## What changed vs the Garmin-first plan

| Area | Garmin-first (superseded) | Muse Athena EEG-first (this scope) |
| --- | --- | --- |
| Primary input | Garmin wrist wearable (HR/HRV/accel) | **Muse S Athena EEG** (4 primary + 4 aux ch, 256 Hz) |
| Sleep insight source | Garmin Connect COTS staging | Direct **EEG features + custom sleep-state**; YASA as candidate/validation (fork open) |
| Gateway | Raspberry Pi | **Powered desktop / mini-PC** recommended (Pi vs mini-PC still open) |
| Data path risk | Watch→Pi transport, foreground battery | Athena BLE stream, overnight battery (~10 h), local buffering/replay |
| Control rollout | Fire therapy once algorithm validated | **Shadow mode first → restricted active** only after quality/latency/classification gate |
| Timing problem | Detect deep-sleep onset (classification) | Predict **before** deep-sleep onset (lead-time prediction) on a sparse frontal montage |
| Reports | Stage timestamps + therapy timestamps | Above **+** time-per-stage, deep-sleep qualification (cycles, duration, slow-wave amplitude/depth) |

## Guiding architecture (local closed loop)

`Muse Athena → powered gateway (desktop/mini-PC) → acquisition & quality-control service → EEG feature & state engine → therapy-decision service → Restiv controller.`

Full-resolution data stored locally; cloud only for post-night upload/analysis/training/admin. The time-critical loop must not depend on the internet. Restiv receives compact state/control messages, never the raw 256 Hz stream. EEG is the high-resolution **reference** for baseline + the first 3–5 therapy nights; a lower-burden EEG-calibrated autonomic model (HRV/Garmin/motion) is a later, Study-2 track — not a Study-1 replacement.

---

## Phase 0 — Muse Athena technical spike + commercial/architecture lock (NEW, ~2–3 weeks)

**Objective.** De-risk the pivot before any build commitment and produce the inputs needed to re-estimate Phases 1–3. Explicitly requested in the handoff's "next steps."

**Activities.** Review Muse SDK / Muse Direct / licensing + sample apps; connect to a live Athena; record raw EEG with device + host timestamps; verify channels/rates/units/battery telemetry; prove a replay pipeline; measure packet loss and reconnection; test the Muse→gateway transport; decide gateway platform (Pi vs mini-PC); confirm **SDK licensing and raw/derived-data rights**.

**Wave-feature streaming spike (near-real-time trigger primitives).** Build a thin **streaming wrapper** around YASA's `bandpower` and `sw_detect` (which are near-real-time capable — unlike YASA's non-causal stage classifier — but ship as batch functions on an array, not as a live runner). Feed rolling windows from the live Athena stream, compute **delta/slow-wave power** and **slow-wave events** continuously, and **characterize detection latency**. **Re-tune the amplitude thresholds for the frontal Athena montage** (YASA defaults are calibrated for adult central derivations) and add **online artifact rejection** (movement / poor contact). See [[biopredictx-alpha2-yasa-viability.md]].

- **Timing floor to design against:** band power needs a window **≥ ~4 s** to resolve delta (YASA rule: window ≥ 2 × 1/f_low; default `win_sec=4`); slow-wave event detection needs a **~5–10 s rolling buffer** and returns events with **~1–3 s lag**. This sets the earliest the therapy trigger can fire — acceptable because deep-sleep transitions unfold over minutes, not seconds.

**Deliverables.** A definitive **sensor/data dictionary from actual SDK outputs**; a raw session-file format + processed-data schema + Restiv state-packet contract (draft); a gateway-platform recommendation; a battery/continuity qualification protocol; a **wave-feature latency & threshold feasibility memo** (measured delta-power/slow-wave detection latency on frontal Athena data, with re-tuned thresholds and the chosen window sizes); and a **re-estimate of Phases 1–3** schedule/staffing/budget.

**Exit gate.** Live Athena connect + raw recording + valid timestamps + battery reporting + replay + packet-loss characterization demonstrated; streaming `bandpower`/`sw_detect` producing delta-power + slow-wave events in near-real-time with measured latency and frontal-tuned thresholds; licensing/rights confirmed sufficient; gateway platform chosen.

**Relative size:** S–M (~2–3-week time-boxed spike). The wider window reflects the real work here — SDK licensing, the frontal-montage threshold tuning, the streaming wrapper, and latency characterization are more than a week. Blocks firm estimation of everything downstream.

**Risks (Phase 0).**

- **P0-R1 — No / late access to the Muse SDK.** Athena's real-time streams require the licensed Muse SDK (Interaxon partner/developer access). If access isn't granted, or is delayed, the entire spike is blocked and nothing downstream can be estimated. *Mitigation: initiate the SDK/licence request immediately and confirm access is in hand **before** committing the spike window; use the Muse Direct desktop tool as an interim path to characterize the signal while SDK access is pending; confirm derived-data/IP rights at the same time (links to R2').*
- **P0-R2 — Unknown implementation language / tech stack, dependent on team familiarity.** The SDK dictates which languages and platforms are viable (e.g. iOS/Android vs desktop bindings). If the only supported path is a stack the team isn't fluent in, ramp-up time and delivery risk increase. *Mitigation: confirm the SDK's supported languages/bindings in week 1; choose the stack the team knows best **among those the SDK supports**; if a less-familiar stack is forced, budget explicit ramp-up and reflect it in the re-estimate.*
- **P0-R3 — Gateway hardware undecided (mini-PC vs Raspberry Pi), OS and capabilities unknown.** We don't yet know which OS to run, what compute is needed for real-time processing/replay, or whether the required BLE version/stack is supported on the candidate hardware. Choosing wrong forces rework in Phase 2. *Mitigation: derive the hardware requirements from the SDK/OS support matrix (supported OS, BLE version, continuous-power need, CPU for processing + replay); verify BLE + OS compatibility on the candidate device during the spike; recommend the platform at the exit gate.*

---

## Phase 1 — EEG acquisition, sleep-state engine & therapy-decision logic (shadow) — highest risk

**Objective.** Prove the core premise on the new architecture: that Athena EEG can be acquired at quality overnight, that a **custom sleep-state** can be inferred causally, and that a therapy decision can be made **before** deep-sleep onset — all validated in **shadow mode** against synchronized EEG+Restiv records.

**Tangible asset.** On the chosen gateway: acquisition + quality-control service, an EEG feature/state engine, and a therapy-decision service running in **shadow** (recommendations logged, no live actuation), plus a replay harness that runs recorded nights through the same interfaces used live. Written, clinician-reviewed sleep-state + timing definition.

*Working assumption (features vs stages):* the live trigger is **EEG-feature/event-driven** — built on the Phase-0 streaming primitives (delta/slow-wave power trend + slow-wave event rate/amplitude from YASA `bandpower`/`sw_detect`), fired as deep-sleep emerges — **not** a discrete AASM stage classifier. YASA's stage classifier + slow-wave detection are retained for **offline scoring, deep-sleep qualification reporting, and as the validation reference** (see [[biopredictx-alpha2-yasa-viability.md]]). The exact trigger signature and threshold are the clinician sign-off item (Open Decisions #2, #3, #6).

**Requirements addressed.** FR-E1 (connect/configure Athena), FR-E2 (ingest/persist native-resolution + dual timestamps), FR-E3 (fault/quality detection), FR-E5 (signal-quality gate), FR-E6 (feature set + state probabilities), FR-E7 (replay through live interfaces), FR-E8 (versioned shadow recommendations + confidence/reason codes), FR-E10 (time-sync store within tolerance); NFR-E: local real-time, latency/clock-drift measurement, immutable raw retention + reproducible reprocessing, no silent data loss.

**Study steps.** Bench/staff qualification → ≥2 baseline nights (Restiv off) → therapy nights 1–3 (fixed Restiv protocol, EEG algorithm in shadow).

**Objection retired.** "You can't infer a usable sleep state from Athena and decide *before* deep sleep in real time overnight."

**Exit gate.** Signal quality, end-to-end latency/clock drift, and classification acceptable against synchronized reference; sleep-state + required **lead-time/tolerance** signed off by a client clinician/sleep-science reviewer; shadow recommendations logged with confidence + reason codes.

**Relative size:** L–XL (the scientific + real-time risk lives here). Staging-engine fork (YASA vs custom vs YASA-as-validation) must be decided at or before this phase — see Open Decisions.

**Risks (Phase 1).**

- **P1-R1 — Before-deep-sleep prediction may not achieve usable lead time** on a sparse frontal EEG montage (this is prediction, not classification). The core premise could fail here. *Mitigation: shadow-mode validation against synchronized reference; renegotiate required lead-time/tolerance with the clinician if needed (R1').*
- **P1-R2 — No in-house sleep-science expertise to define/validate the custom sleep-state.** Without a qualified reviewer, "correct" has no owner. *Mitigation: named client-side clinical/sleep-science reviewer; their sign-off is part of the Phase 1 exit gate (R10').*
- **P1-R3 — Staging-engine fork unresolved (YASA vs proprietary vs YASA-as-validation).** Blocks both the build approach and the estimate. *Mitigation: force the decision at or before Phase 1 (see Open Decisions); links to [[biopredictx-alpha2-huggingface-model-approach]].*
- **P1-R4 — Over-trusting Muse-derived stages as ground truth.** *Mitigation: treat EEG as reference not truth; clinician review; retain raw for reprocessing (R7').*
- **P1-R5 — Low-frequency artifact mistaken for delta / poor overnight contact** corrupts state inference. *Mitigation: signal-quality gate before any state/control output (FR-E5); do not equate all delta power with restorative N3.*
- **P1-R6 — Timestamp drift breaks EEG↔therapy cause/effect** in the analysis. *Mitigation: dual device+host timestamps, <1 s sync target, drift characterized (FR-E10, R6').*
- **P1-R7 — Premature algorithm complexity** chasing noisy features. *Mitigation: shadow-first, fixed-protocol early nights; add complexity only after gates pass (R9').*

---

## Phase 2 — Gateway acquisition service, overnight robustness & operator view

**Objective.** Harden the now-proven acquisition path into a dependable overnight service on the powered gateway, with the transport to Restiv and an operator status view.

**Tangible asset.** A gateway acquisition service with buffering, auto-reconnection, local storage, real-time processing, the Restiv message transport, and an operations view of signal health / battery / connection / session state. Sustained, validated overnight capture.

**Requirements addressed.** FR-E4 (auto-recover, preserve/document gaps), FR-E9 (bounded commands to Restiv + ack/actual-state/faults — transport), FR-E12 (operations views), FR-E14 (secure resumable post-session upload with no impact on local loop); NFR-E: overnight continuity/recovery under the selected config, deterministic safe state after fault, modular sensor abstraction.

**Acceptance targets (from handoff — verify/tune).** Usable runtime ≥ full session (design 10–12 h w/ setup margin); EEG completeness ≥95%; raw file recovery 100%; time-sync error <1 s (characterize drift); longest unrecovered loss <30–60 s; end-of-night reserve ≥15–20%; raw volume ~100–250 MB/night allowance until measured.

**Objection retired.** "It works on the bench but not reliably across a full unattended night."

**Exit gate.** Overnight acceptance targets met under the selected study config; safe-state and recovery demonstrated; operator view usable by trained staff without developer intervention.

**Relative size:** M–L.

**Risks (Phase 2).**

- **P2-R1 — Athena battery below a full night** under custom continuous streaming. *Mitigation: Phase 0 battery protocol; Phase 2 overnight acceptance targets (≥15–20% end-of-night reserve); setup-margin design of 10–12 h (R3').*
- **P2-R2 — BLE interruption with no buffering invalidates the night.** *Mitigation: local buffering + auto-reconnect that preserves and documents gaps (FR-E4); packet-loss already characterized in Phase 0 (R4').*
- **P2-R3 — Gateway/desktop failure or OS sleep-state silently stops the loop.** *Mitigation: powered mini-PC over battery-backed mobile OS; disable OS suspend; deterministic safe state on fault (R8').*
- **P2-R4 — Overnight electrode contact degradation** reduces usable data late in the night. *Mitigation: continuous signal-quality metric + machine-readable missingness; no silent data loss (FR-E3/E5).*
- **P2-R5 — Acceptance targets not met under real unattended conditions** (completeness ≥95%, sync <1 s, recovery 100%). *Mitigation: overnight qualification runs against the targets before the phase gate; tune or renegotiate targets with evidence.*

---

## Phase 3 — Therapy controller, Operations Portal, backend & controlled activation

**Objective.** Assemble the full Alpha 2.0 study system and enable **restricted closed-loop activation** behind the safety model, with the expanded reporting the pivot requires.

**Tangible asset.** Integrated system running an end-to-end study session: portal-managed users/devices, live Athena capture, quality-gated state inference, therapy fired on the **real** Restiv device under restricted-active mode, tiered data stored/exportable, full audit logs, expanded stage/deep-sleep-qualification reports.

**Requirements addressed.** FR-E9 (full controller — command + ack + actual state + parameters + faults), FR-E11 (study configs: baseline / fixed-treatment / shadow / restricted-control), FR-E13 (store/export raw/processed/state/command/audit with pseudonymous IDs); expanded portal reporting (time-per-stage; deep-sleep cycles, per-cycle duration, slow-wave amplitude/depth); NFR-E: least-privilege access, de-identification, encryption, full auditability, safe-state guarantees.

**Control & safe-state model.** Modes = replay / live-acquisition-only / shadow / restricted-active / manual-engineering. Early allowed actions = start, maintain, pause, resume, taper/stop. Quality gate + lockout/dwell + hard exposure limits + fault→safe-state + command acknowledgement + manual override + full version control. Optional study nights 4–5 = restricted control **only if** the Phase 1/2 gate passed.

**Objection retired.** "The pieces don't come together into a system researchers can safely run with the device live."

**Exit gate.** A researcher runs a full session via the portal; therapy fires on the real device under the quality gate and lockouts; fault path forces safe state (therapy off); permissions enforced; raw/processed/audit data exportable; expanded reports produced.

**Relative size:** L (largest raw volume; safety/audit surface larger than the Garmin-first Phase 3).

**Risks (Phase 3).**

- **P3-R1 — Restricted-active safety: a wrong live command reaches a sleeping participant** if the quality gate or lockouts fail. This is the highest-consequence risk once the device is live. *Mitigation: shadow-first rollout; quality gate + lockout/dwell + hard exposure limits + fault→safe-state (therapy OFF) + command acknowledgement + manual override; restricted control only after the Phase 1/2 gate passes.*
- **P3-R2 — Control logic still "TBD — clinician input required"**, so Phase 3 cannot be fully specified or estimated until sign-off. *Mitigation: obtain clinician sign-off on modes/actions/gates/lockouts before locking Phase 3 scope (Open Decision #3).*
- **P3-R3 — Data-protection & audit surface is larger than Garmin-first** (raw/processed/state/command/audit tiers, pseudonymous IDs, encryption, least-privilege). Gaps create compliance exposure. *Mitigation: de-identification + full auditability + encryption designed in, not bolted on (FR-E13, NFR-E).*
- **P3-R4 — Integration risk: the proven pieces don't assemble into a system researchers can safely run.** *Mitigation: end-to-end session rehearsal via the portal on the real device before handoff; fault-path and permission testing as gate criteria.*

---

## Later / Study 2 track — EEG-calibrated multimodal autonomic control (not in Alpha 2.0 build)

Once the EEG loop is validated, translate the EEG-derived state model into a lower-burden autonomic model (HRV, HR, respiration, motion, temperature, longitudinal history), running **in shadow while Athena remains the reference**. Candidate parallel prototype platforms flagged by the client: Garmin Vívóactive 5 (conditional on Health SDK / IBI access — see [[biopredictx-alpha2-sleep-algorithm-selection]]) and Samsung Galaxy Watch (raw PPG/ECG access). FR-E15 keeps the Restiv control contract stable so these can be added without rework.

---

## Requirement → phase coverage (FR-E / NFR-E)

| Requirement | P0 | P1 | P2 | P3 |
| --- | --- | --- | --- | --- |
| FR-E1 Connect/configure Athena | spike | ● | | |
| FR-E2 Ingest/persist native res + timestamps | proto | ● | | |
| FR-E3 Fault/quality detection | | ● | ● | |
| FR-E4 Auto-recover / preserve gaps | | | ● | |
| FR-E5 Signal-quality gate | | ● | | ● |
| FR-E6 Feature set + state probabilities | | ● | | |
| FR-E7 Replay through live interfaces | spike | ● | | |
| FR-E8 Shadow/active recommendations + confidence | | ● shadow | | ● active |
| FR-E9 Bounded commands to Restiv + ack | | | ● transport | ● controller |
| FR-E10 Time-sync store within tolerance | proto | ● | ● | |
| FR-E11 Study configs (baseline/fixed/shadow/restricted) | | begin | | ● |
| FR-E12 Operations views | | | ● | ● |
| FR-E13 Store/export raw/processed/state/audit | | | begin | ● |
| FR-E14 Secure resumable upload | | | ● | |
| FR-E15 Future autonomic/wearable streams | design principle | | | contract |
| NFR-E local real-time, no internet dependency | | ● | ● | |
| NFR-E latency / clock-drift within tolerance | measure | ● | ● | |
| NFR-E overnight continuity/recovery | | | ● | |
| NFR-E deterministic safe state | | | ● | ● |
| NFR-E modular sensor abstraction | design | | | ● |
| NFR-E immutable raw + reproducible reprocessing | | ● | | ● |
| NFR-E least-privilege / de-id / encryption / audit | | | | ● |
| NFR-E no silent data loss (machine-readable missingness) | | ● | ● | |

## Phase summary

| Phase | Focus | Tangible asset | Risk retired |
| --- | --- | --- | --- |
| 0 | Athena spike + commercial/architecture lock | Data dictionary, contracts, gateway decision, re-estimate | Do we understand what Athena exposes + our rights to it? |
| 1 | EEG acquisition + sleep-state engine + decision (shadow) | Quality-gated shadow loop + clinician-signed state/timing def | Can we infer state + decide *before* deep sleep, overnight? |
| 2 | Gateway robustness + operator view | Reliable overnight capture + Restiv transport | Does it survive a full unattended night? |
| 3 | Controller + portal + backend + restricted activation | Integrated study system firing therapy safely on the real device | Do the pieces form a safe, usable whole? |

## Indicative sizing (pre-spike — NOT a commitment)

Firm numbers are withheld until the Phase 0 spike. Directionally vs the ~$210K / ~2,470 h / ~23-week Garmin-first plan: Phase 1 grows (EEG signal-quality, custom state engine, shadow-mode instrumentation, baseline nights, clinician loop); Phases 2–3 grow modestly (quality gating, safe-state/lockout logic, audit tiers, expanded reporting) while shedding the Garmin foreground-battery/transport problems. Net expectation: **similar-to-higher** total, with more of the effort and risk front-loaded into Phase 1. Confirm at the Phase 0 gate.

## Open decisions requiring client / clinician sign-off (gate the estimate)

1. **Staging-engine fork:** adopt **YASA** (Lindsey's email) / build a proprietary feature-state engine (Docs 1 & 2) / use YASA as a validation reference only. Different cost + timeline. See [[biopredictx-alpha2-huggingface-model-approach]].
2. **Timing definition:** required **lead time before deep sleep** + tolerance — this is a prediction problem, not classification. Clinician input required.
3. **Control logic:** modes/actions/gates/lockouts are drafted in Doc 2 but marked "TBD — clinician input required." Needs sign-off before Phase 3 estimation.
4. **SDK licensing & raw/derived-data rights:** must be settled in Phase 0 before architecture lock.
5. **Gateway platform:** Raspberry Pi vs desktop/mini-PC.
6. **"Features vs stages" tension:** Doc 1's direct-EEG-features framing vs Lindsey's "customized sleep-state + before-deep-sleep timing." Reconcile before over-investing in either.
7. **Scientific caution:** Restiv rotation frequency ≠ EEG slow-wave frequency; do not design the controller to "chase" the EEG frequency — treat as a testable hypothesis. Not all delta power = restorative N3.

## Cross-phase risk roll-up (R-prime register)

Master register that the per-phase **Risks** sections above draw on (P1–P3 cite these R' IDs; Phase 0's P0-R1…R3 are new and specific to the spike). Supersedes/extends the Garmin-first register; see [[biopredictx-alpha2-risks]].

| # | Risk | Mitigation |
| --- | --- | --- |
| R1' | **Before-deep-sleep prediction on a sparse frontal EEG montage** may not achieve usable lead time. | Phase 1 shadow-mode validation vs synchronized reference; renegotiate lead-time/tolerance with clinician if needed. |
| R2' | **SDK/licensing limits & derived-data rights** could constrain architecture or IP. | Settle in Phase 0 before lock. |
| R3' | **Athena battery below full night** under custom streaming. | Phase 0 battery protocol; Phase 2 overnight acceptance targets. |
| R4' | **BLE interruption / no buffering** invalidates data. | Local buffering + auto-reconnect (FR-E4); packet-loss characterized in Phase 0. |
| R5' | **Overnight contact degradation / low-freq artifact mistaken for delta.** | Quality gate (FR-E5); do not equate all delta with N3. |
| R6' | **Timestamp drift** breaks cause/effect between EEG and therapy. | Dual timestamps + <1 s sync target; characterize drift (FR-E10). |
| R7' | **Over-trusting Muse-derived stages as ground truth.** | Treat EEG as reference, not truth; clinician review; retain raw for reprocessing. |
| R8' | **Desktop/gateway failure or OS sleep** interrupts the loop. | Powered mini-PC recommendation; deterministic safe state on fault. |
| R9' | **Premature algorithm complexity.** | Shadow-first, fixed-protocol early nights; add complexity only after gates. |
| R10' | **No in-house sleep-science expertise** (carried from R2 original). | Named client-side clinical reviewer; sign-off is part of the Phase 1 gate. |

## Provenance

Direction: [[biopredictx-alpha2-muse-athena-eeg-pivot]] (email "Info from BPTx", 21 Jul 2026; Allison London Brown, reframed by Lindsey Wilkinson). Supersedes [[biopredictx-alpha2-phased-proposal]]. Source docs: Muse Direct & SDK Overview v1.0; Muse S Athena Technical Handoff v1.0; Wearable Sensor & Data-Access Assessment v1.0.
