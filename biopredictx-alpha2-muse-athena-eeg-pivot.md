---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-phased-proposal]]"
  - "[[biopredictx-alpha2-risks]]"
  - "[[biopredictx-alpha2-sleep-algorithm-selection]]"
  - "[[biopredictx-alpha2-huggingface-model-approach]]"
  - "[[biopredictx-fdc-alpha-2-0]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — Muse Athena EEG Pivot (Handoff, Overview & Wearable Assessment)

Captured context from the client hardware/architecture pivot: Alpha 2.0 moves from a **Garmin-first** wearable architecture to a **Muse S Athena EEG-first** architecture. This note consolidates the three documents BioPredicTx sent and the email thread that frames them.

## Status & provenance

- Source: email thread **"Info from BPTx"**, 21 Jul 2026. From Allison London Brown (BioPredicTx), cc Lindsey Wilkinson (BioPredicTx); to Liam Shanahan, Rodrigo Fernandez, Allen Williams (DualBoot).
- Three attachments, all **agent-generated** by the client ("research and then digestion by my agent"):
  1. Muse Direct & SDK — Direct EEG Control Overview v1.0
  2. Muse S Athena Integration & EEG-Driven Therapy Technical Handoff v1.0
  3. Wearable Sensor and Data-Access Technical Assessment v1.0
- **Caveat (Lindsey, 21 Jul):** these documents are **not** to be treated as project definition or requirements yet — pending her review and internal discussion. This note is captured context, not locked scope.

## Headline change: Garmin → Muse Athena (EEG-first)

Per Lindsey's authoritative email:

- **Original intent (FDC):** leverage COTS sleep-staging via Garmin to control Restiv therapy timing in a closed loop. Muse EEG was only an *accessory* for user testing via the consumer app (deep-sleep duration/depth/fragmentation) — no integration with the software project.
- **Updated plan:** integrate an at-home EEG headband — the **Muse S Athena** — as the primary input. Requires a **customized definition of sleep state**. Critical timing requirement: therapy must start **before** the user enters deep sleep.
- Report scope via the web portal expands once the Muse consumer app is retired.

| Area | Previous | Current |
| --- | --- | --- |
| Hardware selection | Garmin (wrist wearable) | Muse Athena (EEG) |
| Sleep insights | Garmin Connect (COTS) | Open-source EEG analyzers — **YASA** |
| Therapy control logic | Sleep-stage triggers (COTS) | **TBD — clinician inputs required** |
| User-test reports | Sleep-stage timestamps; therapy-active timestamps; raw exports | Above **plus** total time per stage (Awake/Light/Deep/REM); deep-sleep qualification (# cycles, duration per cycle, slow-wave amplitude/depth) |

## Document 1 — Muse Direct & SDK / Direct EEG Control Overview

- Recommends using **Muse Direct** (desktop tool) first for acquisition, visualization, recording and replay — the fastest way to characterize what Athena actually exposes (channels, rates, units, processed values, battery) before locking architecture. Then use the **Muse SDK** to build the BioPredicTx app for programmatic control (Muse Direct alone can't control Restiv).
- Argues the first algorithm should drive control off **direct EEG features** (signal validity, delta magnitude, slow-oscillation, persistence, fragmentation, spatial agreement, response-to-therapy) rather than N1/N2/N3/REM stage labels; stages retained only as secondary annotation.
- **Tension:** this "features-not-stages" framing partly conflicts with Lindsey's direction (a *customized sleep-state* definition + timing framed around "before deep sleep"). Do not over-invest in it until reconciled.

## Document 2 — Muse S Athena Technical Handoff (redline of the phased proposal)

The most actionable document. Explicitly written as an **update to DualBoot's existing "Alpha 2.0 — Phased Implementation" proposal** ([[biopredictx-alpha2-phased-proposal]]), including a find/replace appendix of Garmin→Athena language.

**Decision locked:** Muse S Athena is the primary real-time physiological input; Garmin drops to at most a secondary longitudinal/autonomic source.

**Product strategy:** EEG is the high-resolution *reference* for baseline + the first 3–5 therapy nights. Synchronized EEG+Restiv records characterize sleep state, delta emergence, N3 stability, fragmentation, arousal, treatment response. Later, translate the EEG-derived state model into a lower-burden **autonomic model** (HRV, HR, respiration, motion, temperature, longitudinal history) — described as "EEG-calibrated multimodal autonomic control," not "replace EEG with one HRV score."

**Architecture (local closed loop):**

> Muse Athena → powered desktop / mini-PC gateway → acquisition & quality-control service → EEG feature & state engine → therapy-decision service → Restiv controller.

- Full-resolution data stored locally; cloud only for post-night upload/analysis/training/admin. Time-critical loop must not depend on internet.
- Restiv receives **compact state/control messages**, not the raw 256 Hz stream.
- Earliest nights: **fixed Restiv protocol + EEG algorithm in shadow mode**; restricted closed-loop only after signal/latency/classification acceptable.
- Recommends a **desktop-class gateway over the Raspberry Pi** (continuous power, no mobile-OS suspension, local CPU for processing/replay). Pi-vs-mini-PC left open.

**Phase rewrite:**

| Phase | Current (Garmin-first) | Required update (Athena EEG-first) |
| --- | --- | --- |
| 1 | Detect deep-sleep onset from wearable; choose Garmin vs Muse | Lock Athena as primary; validate EEG acquisition, signal quality, feature extraction, sleep-state inference, timing, therapy-decision logic. Spikes: Athena SDK/desktop, live BLE stream, replay pipeline, battery/runtime, channel mapping, timestamp validation, Muse→gateway transport |
| 2 | Minimal watch capture + watch→Pi transport | Athena acquisition service on a powered desktop/mini-PC gateway: buffering, reconnection, local storage, real-time processing, Restiv message transport, operator status view |
| 3 | Controller, portal, backend, device integration | Retain, but add EEG state packets, algorithm confidence, data-quality gating, shadow/active modes, treatment lockouts, raw/processed data tiers, full audit logs |

**Athena technical baseline (verify against actual SDK/firmware):** 4 primary EEG channels + 4 amplified auxiliary; 256 Hz; 14-bit; accelerometer ~52 Hz; PPG + fNIRS present; 150 mAh battery, ~10 h advertised, ~3 h charge. Public specs are engineering-verification items, not binding requirements.

**Revised functional requirements (FR-E1…E15):**

- FR-E1 Connect/identify/configure an approved Athena via the licensed SDK/desktop tools.
- FR-E2 Ingest & persist all required streams at native resolution with device + host timestamps.
- FR-E3 Detect missing packets, stream interruption, channel failure, poor contact, low battery, acquisition faults.
- FR-E4 Auto-recover from connection loss; preserve/document gaps.
- FR-E5 Versioned signal-quality metrics; enforce a quality gate before state/control output.
- FR-E6 Compute the predefined EEG feature set + stage/state probabilities at specified cadences.
- FR-E7 Replay recorded files through the same processing/decision interfaces used live.
- FR-E8 Versioned shadow or active therapy recommendations with confidence + reason codes.
- FR-E9 Send bounded commands to Restiv; capture ack, actual state, parameters, faults.
- FR-E10 Synchronize & store Athena/gateway/algorithm/Restiv events within validated timing tolerance.
- FR-E11 Support baseline, fixed-treatment, shadow-mode, restricted-control study configs.
- FR-E12 Operations views: signal health, battery, connection, session/device state, faults.
- FR-E13 Store/export raw, processed, state, command, audit data with pseudonymous IDs.
- FR-E14 Secure post-session upload with resumable interrupted uploads; no impact on local loop.
- FR-E15 Support future Garmin/HRV/wearable streams without changing the Restiv control contract.

**Revised nonfunctional (NFR-E1…E9):** local real-time operation without internet dependency; measurable end-to-end latency/clock drift within tolerance; overnight continuity/recovery validated under the selected config; deterministic safe state after any fault; modular sensor abstraction (EEG can later supervise an autonomic model); immutable raw retention + reproducible reprocessing via versioned algorithms; least-privilege access, de-identification, encryption, full auditability; no silent data loss (machine-readable missingness); research workflow usable by trained staff without developer intervention.

**Control & safe-state model:** modes = replay / live-acquisition-only / shadow / restricted-active / manual-engineering. Early allowed actions = start, maintain, pause, resume, taper/stop. Quality gate + lockout/dwell + hard exposure limits + fault→safe-state + command acknowledgement + manual override + full version control.

**Baseline & first nights:** bench/staff qualification → ≥2 baseline nights (Restiv off) → therapy nights 1–3 (fixed protocol, shadow mode) → optional nights 4–5 restricted control if gate passed → later autonomic/HRV model in shadow while Athena remains active.

**Battery/overnight acceptance targets:** usable runtime ≥ full session (design 10–12 h w/ setup margin); EEG completeness ≥95%; raw file recovery 100%; time-sync error <1 s (characterize drift); longest unrecovered loss <30–60 s; end-of-night reserve ≥15–20%. Raw EEG volume ~100–250 MB/night practical allowance until measured.

**Key risks (handoff):** SDK/licensing limits & derived-data rights; battery below full-night under custom streaming; BLE interruption/no buffering invalidating data; overnight contact degradation; low-frequency artifact mistaken for delta; over-trusting Muse-derived stages as ground truth; timestamp drift breaking cause/effect; controller reacting too often to noisy features; desktop failure/sleep-state; premature algorithm complexity. Aligns with [[biopredictx-alpha2-risks]].

**Next steps for DualBoot (from handoff):** review Muse SDK/desktop/license + sample apps; replace Garmin-first assumptions in Phase 1/2 scope; propose gateway platform + local Restiv transport (keep vs replace Pi); produce a **1-week technical spike plan** (live Athena connect, raw recording, timestamps, battery reporting, replay, packet-loss); build a definitive sensor/data dictionary from actual SDK outputs; define raw session-file format, processed-data schema, Restiv state-packet contract; create the overnight battery/continuity qualification protocol; re-estimate Phase 1–3 schedule/staffing/budget after the spike; identify all science/algorithm decisions requiring client or external sleep-science reviewer sign-off.

**Scientific caution:** Restiv applies mechanical or field-rotation stimulation. The device rotation frequency and the endogenous EEG slow-wave frequency are **different physical variables** — a numeric match does not mean the device should "chase" the EEG frequency. Any such relationship is a testable hypothesis, not a design assumption. Do not equate all low-frequency (delta) power with restorative N3.

## Document 3 — Wearable Sensor & Data-Access Technical Assessment

Landscape/context across Muse, Garmin Vívóactive 5, Fitbit/Google, Apple Watch, Samsung Galaxy Watch, Oura, WHOOP.

**Thesis:** the deciding factor is not which sensors a device has, but whether BioPredicTx can obtain **time-synchronized, granular, quality-scored data with predictable latency and stable commercial rights**. Internal sampling ≠ exported resolution (e.g. a ring samples PPG at hundreds of Hz but exposes only 5-min HRV). Recommended long-term architecture matches Doc 2: **EEG-calibrated multimodal autonomic control**.

| Platform | Recommended role | Real-time control eligibility |
| --- | --- | --- |
| Muse | Research reference / EEG supervisory sensor | **Yes — use now** |
| Garmin Vívóactive 5 | Study 2 autonomic recorder; possible future controller | Conditional on Health SDK / IBI access |
| Samsung Galaxy Watch | Raw-sensor algorithm-development platform (raw PPG 25 Hz cont. / 100 Hz on-demand; ECG 500 Hz) | High potential — begin parallel prototype |
| Apple Watch | Commercial ecosystem / monitoring; later qualified control | Conditional (HealthKit HRV is SDNN only, no continuous RR) |
| Oura | Overnight phenotype / personalization | No via public API |
| WHOOP | Recovery / responder phenotype | No via public API |
| Fitbit / Google | Broad consumer monitoring | Low — secondary |

Also provides a reference ingestion architecture (device adapter → raw immutable store → time-sync → signal-quality → feature engine → state estimator → therapy policy → audit log), a minimum data contract, Study 2 acquisition priorities, and vendor diligence questions.

## Open questions & tensions to resolve before locking scope

- **Staging engine fork:** Lindsey's email names **YASA** (open-source). Docs 1 & 2 describe a **custom** feature/state engine and never mention YASA. Decide: adopt YASA / build proprietary / use YASA as validation reference. Different cost + timeline. Relates to [[biopredictx-alpha2-huggingface-model-approach]] and [[biopredictx-alpha2-sleep-algorithm-selection]] (Garmin selection now superseded).
- **Control logic:** marked "TBD — clinician input required" by Lindsey, but Doc 2 already specifies modes/actions/gates/lockouts. Specifics need clinician sign-off before estimation.
- **Timing = prediction, not classification:** "start therapy *before* deep sleep" on a sparse frontal montage is a lead-time prediction problem. Need required lead time + tolerance from the client.
- **SDK licensing & raw-data rights:** unconfirmed; must be settled before architecture lock.
- **Gateway platform:** Raspberry Pi vs desktop/mini-PC still open.
- **FDC:** referenced as the original design basis but not yet obtained in full — see [[biopredictx-fdc-alpha-2-0]].

## Source documents

- `BioPredicTx_Muse_Direct_SDK_Direct_EEG_Control_Overview_v1_0.docx`
- `BioPredicTx_DualBoot_Muse_Athena_Technical_Handoff_v1_0.docx`
- `BioPredicTx_Wearable_Sensor_and_Data_Access_Technical_Assessment_v1_0.docx`
- Email thread "Info from BPTx", 21 Jul 2026 (Allison London Brown; Lindsey Wilkinson reframe).
