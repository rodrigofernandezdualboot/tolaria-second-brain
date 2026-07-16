---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-risks]]"
  - "[[biopredictx-alpha2-requirements-estimation]]"
  - "[[biopredictx-tshirt-sizing]]"
  - "[[biopredictx-scope-change-client-summary]]"
status: Active
---
# BioPredictX Alpha 2.0 — Phased Implementation Proposal

## Phase 1 — Algorithm definition & validation on a Raspberry Pi simulation

**Objective.** Prove the core premise before committing to the full build: that the system can detect the onset of deep sleep from wearable signals in real time and trigger therapy within the required window. Define and validate the sleep-staging and therapy-decision algorithm.

**Tangible asset delivered.** A Raspberry Pi simulation — the candidate sleep-staging algorithm running on the Pi against sensor data \(recorded replay and/or live\), applying the therapy-trigger logic and logging each therapy ON/OFF decision with its timing against ground-truth sleep stages. No live magnet hardware; the value is the algorithm executing and the logs it produces. Plus a validated algorithm definition and a written feasibility + wearable decision. This is demonstrable end-to-end on a bench.

**Requirements addressed.** FR-A7 \(analytics / sleep staging — the core\); FR-A8 therapy ON/OFF *decision logic* validated in simulation; FR-A6 ingestion in simulated/replayed form; resolves NFR-A1 \(real-time loop\) and NFR-A2 \(timing\); settles NFR-A3 \(COTS-first vs. custom\). Runs the feasibility spikes for FR-A12 \(Garmin capture\) and FR-A13 \(watch→Pi transport\).

**Key activities.** Select candidate signals and algorithm \(COTS/Garmin staging vs. EEG-based\); implement on the Pi; run the simulation over recorded sleep data \(replay\) and optionally live streams from Garmin and Muse; instrument detection latency vs. expert-scored ground truth; test therapy parameters and thresholds; run the transport/capture spikes \(Pi-as-BLE-peripheral or Wi‑Fi HTTP; Connect IQ sensor sampling\).

**Objection it retires.** "You can't detect deep sleep fast enough to trigger therapy in time — and the Garmin data path may not even work."

**Success criteria / exit gate.** Detection latency documented against ground truth; a clear verdict on *before* vs. *at/just-after* deep-sleep onset \(NFR-A2 renegotiated with the client if needed\); wearable decision locked \(Garmin vs. Muse\) with the transport approach proven on the bench; validated algorithm spec handed off.

**Contingency.** If "before onset" proves infeasible, relax NFR-A2 to "at onset" with client sign-off. If Garmin can't stream in real time, switch to Muse EEG — which reshapes the Phase 2 capture work. Better to learn this here, cheaply.

**Relative size:** L–XL \(highest *risk*, focused build\).

---

## Phase 2 — Capture app, device configuration & live ingestion \(parallel tracks\)

**Objective.** Build the now-proven data path end-to-end into the Pi, plus the device-configuration controls and the portal's UX — running two tracks in parallel now that the wearable and algorithm are locked.

**Tangible asset delivered.** Live sensor data streaming from a minimal wearable app into the Raspberry Pi in real time; a working device-configuration capability \(set Restiv therapy parameters — magnet angle, motor acceleration, rotational frequency\); and approved, clickable portal UX designs. Demonstrable: real measurements flowing watch→Pi, live.

**Parallel tracks.** \(a\) Capture + ingestion; \(b\) design/UX + device configuration.

**Requirements addressed.** FR-A12 \(minimal wearable capture app\); FR-A13 \(transport, built on the Phase-1-proven approach\); FR-A6 \(real live ingestion — validation, time-stamping, buffering, streaming\); FR-A2 \(device configuration\); UX design for FR-A1/A3/A4; FR-A5 API groundwork begins.

**Objection it retires.** "Even if the algorithm works in the lab, you can't get live data off the real wearable into the system — and the device can't be configured."

**Success criteria.** Live data sustained overnight from wearable→Pi at the required cadence, within acceptable battery and latency limits; therapy parameters settable and reflected on the device; portal UX approved by your team.

**Relative size:** M–L.

---

## Phase 3 — Therapy controller, Operations Portal & backend integration

**Objective.** Assemble the full Alpha 2.0 system for the test regime — the real therapy controller driving the device, the internal Operations Portal, and the backend/storage/orchestration — integrating the proven pieces. Lower risk, more conventional work.

**Tangible asset delivered.** The complete, integrated Alpha 2.0 system running an end-to-end study session: portal-managed test users and devices, live capture, real-time detection, therapy fired on the *real* device, data stored and exportable, with fault handling.

**Requirements addressed.** FR-A8 \(full therapy controller — command transmission to the Arduino/device, status monitoring\); FR-A1/A3/A4 \(portal user config, device status, data retrieval/reports\); FR-A5 \(API & services backbone\); FR-A9 \(tiered data storage\); FR-A10 \(access control / permissions\); FR-A11 \(operational state machine — sessions, fault/recovery\); NFR-A4 \(administrative controls\); NFR-A6 \(de-identification\); NFR-A7 \(fault / safe state\).

**Objection it retires.** "The pieces don't come together into a usable system your researchers can actually run."

**Success criteria.** A researcher runs a full session end-to-end via the portal; therapy fires on the real device; data is captured, stored, and exportable; the fault path disables therapy safely; permissions are enforced.

**Relative size:** M–L \(largest raw volume, lowest risk — mostly conventional build\).

---

## Requirement → phase coverage

Confirms the plan covers the full Alpha 2.0 scope with nothing orphaned.

| Requirement | Phase 1 | Phase 2 | Phase 3 |
| --- | --- | --- | --- |
| FR-A1 Ops Portal — user config |  | design | build |
| FR-A2 Device configuration |  | ● |  |
| FR-A3 Ops Portal — device status |  | design | build |
| FR-A4 Ops Portal — data retrieval |  | design | build |
| FR-A5 API & services |  | begin | ● |
| FR-A6 Data acquisition / ingestion | simulated | ● live |  |
| FR-A7 Analytics / sleep staging | ● |  |  |
| FR-A8 Therapy control | logic |  | ● controller |
| FR-A9 Data storage |  |  | ● |
| FR-A10 Access control |  |  | ● |
| FR-A11 Operational state machine |  |  | ● |
| FR-A12 Wearable capture app | feasibility | ● build |  |
| FR-A13 Watch→Pi transport | feasibility | ● build |  |
| NFR-A1 Real-time closed loop | ● |  |  |
| NFR-A2 Therapy timing | ● |  |  |
| NFR-A3 COTS-first | ● decision |  |  |
| NFR-A4 Administrative controls |  |  | ● |
| NFR-A5 Modular/extensible | design principle across all phases |  |  |
| NFR-A6 De-identification |  |  | ● |
| NFR-A7 Fault / safe state |  |  | ● |
| NFR-A8 / NFR-A9 | deferred to Beta |  |  |

## Phase summary

| Phase | Focus | Tangible asset | Risk retired |
| --- | --- | --- | --- |
| 1 | Algorithm + feasibility simulation | Pi simulation proving the real-time loop; validated algorithm; wearable decision | Can we detect + trigger in time? |
| 2 | Capture, configuration, ingestion | Live watch→Pi data stream; device config; portal UX | Can we get real live data in? |
| 3 | Controller, portal, backend | Full integrated system running a study session | Do the pieces form a usable whole? |

## Budget & schedule

| Phase | Duration | Estimated hours | Est. Cost \($85/hr\) |
| --- | --- | --- | --- |
| 1 — Algorithm + feasibility simulation | 3–4 wks | 140 | ~$10K |
| 2 — Capture, configuration, ingestion | 7-8 wks | 880 | ~$75K |
| 3 — Controller, portal, backend | 8-11 wks | 1450 | ~$125K |
| **Total** | **~23 weeks** | **2470** | **≈ $210K** |

### Week-by-week

**Phase 1 — Algorithm & feasibility \(3 weeks\)**

- Wk 1 — Select signals/algorithm; stand up the Pi testbed; ingest recorded sleep datasets; set up Garmin + Muse dev environments.
- Wk 2 — Implement candidate sleep-staging on the Pi; build the replay pipeline; define therapy-trigger logic and parameters.
- Wk 3 — Lock the wearable decision; finalize the validated algorithm spec; feasibility report and **gate review**.

**Phase 2 — Capture, configuration, ingestion \(7 weeks\)**

- Wk 1 — Confirm architecture from Phase 1; scaffold the capture app and the Pi ingestion service; UX discovery for the portal.
- Wk 2 — Build the minimal wearable capture app \(sensor sampling\); ingestion validation and time-stamping; portal wireframes.
- Wk 3 — Implement the proven transport; buffering/streaming; device-configuration parameter plumbing; high-fidelity UX.
- Wk 4 — Live end-to-end watch→Pi stream; device-config write path to the controller; UX review.
- Wk 5 — Overnight streaming trials \(cadence, battery, latency\); parameter round-trip; UX sign-off.
- Wk 6 — Harden ingestion; API groundwork; demo the live data path; phase review.
- Wk 7 — Run the transport/capture spikes \(Pi-as-BLE-peripheral / Wi‑Fi HTTP; Connect IQ sampling\); live-stream trials from Garmin and Muse.
- Wk 7 — Measure detection latency vs. ground truth; test parameters and thresholds; before-vs-at-onset analysis.
  
**Phase 3 — Controller, portal, backend \(7 weeks\)**

- Wk 1 — Backend/API scaffolding; data model and storage tiers; portal app shell.
- Wk 2 — Therapy controller: command transmission to the Arduino/device; status monitoring.
- Wk 3 — Operations Portal: user configuration and device status; access control/permissions.
- Wk 4 — Portal data retrieval/reports; storage integration \(raw, processed, logs\).
- Wk 5 — Operational state machine: sessions, fault/recovery, safe therapy shutdown.
- Wk 6 — End-to-end integration; run a full study session on the real device.
- Wk 7 — QA, fault-path testing, de-identification checks, export validation; final demo and handoff.
