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

## Phase 0 — Muse Athena technical spike + commercial/architecture lock (NEW, ~1 week)

**Objective.** De-risk the pivot before any build commitment and produce the inputs needed to re-estimate Phases 1–3. Explicitly requested in the handoff's "next steps."

**Activities.** Review Muse SDK / Muse Direct / licensing + sample apps; connect to a live Athena; record raw EEG with device + host timestamps; verify channels/rates/units/battery telemetry; prove a replay pipeline; measure packet loss and reconnection; test the Muse→gateway transport; decide gateway platform (Pi vs mini-PC); confirm **SDK licensing and raw/derived-data rights**.

**Deliverables.** A definitive **sensor/data dictionary from actual SDK outputs**; a raw session-file format + processed-data schema + Restiv state-packet contract (draft); a gateway-platform recommendation; a battery/continuity qualification protocol; and a **re-estimate of Phases 1–3** schedule/staffing/budget.

**Exit gate.** Live Athena connect + raw recording + valid timestamps + battery reporting + replay + packet-loss characterization demonstrated; licensing/rights confirmed sufficient; gateway platform chosen.

**Relative size:** S (time-boxed spike). Blocks firm estimation of everything downstream.

---

## Phase 1 — EEG acquisition, sleep-state engine & therapy-decision logic (shadow) — highest risk

**Objective.** Prove the core premise on the new architecture: that Athena EEG can be acquired at quality overnight, that a **custom sleep-state** can be inferred causally, and that a therapy decision can be made **before** deep-sleep onset — all validated in **shadow mode** against synchronized EEG+Restiv records.

**Tangible asset.** On the chosen gateway: acquisition + quality-control service, an EEG feature/state engine, and a therapy-decision service running in **shadow** (recommendations logged, no live actuation), plus a replay harness that runs recorded nights through the same interfaces used live. Written, clinician-reviewed sleep-state + timing definition.

**Requirements addressed.** FR-E1 (connect/configure Athena), FR-E2 (ingest/persist native-resolution + dual timestamps), FR-E3 (fault/quality detection), FR-E5 (signal-quality gate), FR-E6 (feature set + state probabilities), FR-E7 (replay through live interfaces), FR-E8 (versioned shadow recommendations + confidence/reason codes), FR-E10 (time-sync store within tolerance); NFR-E: local real-time, latency/clock-drift measurement, immutable raw retention + reproducible reprocessing, no silent data loss.

**Study steps.** Bench/staff qualification → ≥2 baseline nights (Restiv off) → therapy nights 1–3 (fixed Restiv protocol, EEG algorithm in shadow).

**Objection retired.** "You can't infer a usable sleep state from Athena and decide *before* deep sleep in real time overnight."

**Exit gate.** Signal quality, end-to-end latency/clock drift, and classification acceptable against synchronized reference; sleep-state + required **lead-time/tolerance** signed off by a client clinician/sleep-science reviewer; shadow recommendations logged with confidence + reason codes.

**Relative size:** L–XL (the scientific + real-time risk lives here). Staging-engine fork (YASA vs custom vs YASA-as-validation) must be decided at or before this phase — see Open Decisions.

---

## Phase 2 — Gateway acquisition service, overnight robustness & operator view

**Objective.** Harden the now-proven acquisition path into a dependable overnight service on the powered gateway, with the transport to Restiv and an operator status view.

**Tangible asset.** A gateway acquisition service with buffering, auto-reconnection, local storage, real-time processing, the Restiv message transport, and an operations view of signal health / battery / connection / session state. Sustained, validated overnight capture.

**Requirements addressed.** FR-E4 (auto-recover, preserve/document gaps), FR-E9 (bounded commands to Restiv + ack/actual-state/faults — transport), FR-E12 (operations views), FR-E14 (secure resumable post-session upload with no impact on local loop); NFR-E: overnight continuity/recovery under the selected config, deterministic safe state after fault, modular sensor abstraction.

**Acceptance targets (from handoff — verify/tune).** Usable runtime ≥ full session (design 10–12 h w/ setup margin); EEG completeness ≥95%; raw file recovery 100%; time-sync error <1 s (characterize drift); longest unrecovered loss <30–60 s; end-of-night reserve ≥15–20%; raw volume ~100–250 MB/night allowance until measured.

**Objection retired.** "It works on the bench but not reliably across a full unattended night."

**Exit gate.** Overnight acceptance targets met under the selected study config; safe-state and recovery demonstrated; operator view usable by trained staff without developer intervention.

**Relative size:** M–L.

---

## Phase 3 — Therapy controller, Operations Portal, backend & controlled activation

**Objective.** Assemble the full Alpha 2.0 study system and enable **restricted closed-loop activation** behind the safety model, with the expanded reporting the pivot requires.

**Tangible asset.** Integrated system running an end-to-end study session: portal-managed users/devices, live Athena capture, quality-gated state inference, therapy fired on the **real** Restiv device under restricted-active mode, tiered data stored/exportable, full audit logs, expanded stage/deep-sleep-qualification reports.

**Requirements addressed.** FR-E9 (full controller — command + ack + actual state + parameters + faults), FR-E11 (study configs: baseline / fixed-treatment / shadow / restricted-control), FR-E13 (store/export raw/processed/state/command/audit with pseudonymous IDs); expanded portal reporting (time-per-stage; deep-sleep cycles, per-cycle duration, slow-wave amplitude/depth); NFR-E: least-privilege access, de-identification, encryption, full auditability, safe-state guarantees.

**Control & safe-state model.** Modes = replay / live-acquisition-only / shadow / restricted-active / manual-engineering. Early allowed actions = start, maintain, pause, resume, taper/stop. Quality gate + lockout/dwell + hard exposure limits + fault→safe-state + command acknowledgement + manual override + full version control. Optional study nights 4–5 = restricted control **only if** the Phase 1/2 gate passed.

**Objection retired.** "The pieces don't come together into a system researchers can safely run with the device live."

**Exit gate.** A researcher runs a full session via the portal; therapy fires on the real device under the quality gate and lockouts; fault path forces safe state (therapy off); permissions enforced; raw/processed/audit data exportable; expanded reports produced.

**Relative size:** L (largest raw volume; safety/audit surface larger than the Garmin-first Phase 3).

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

## Risks (updated — supersedes/extends the Garmin-first register; see [[biopredictx-alpha2-risks]])

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
