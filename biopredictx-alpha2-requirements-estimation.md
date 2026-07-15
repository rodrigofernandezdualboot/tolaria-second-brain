---
type: Note
belongs_to: "[[biopredictx-fdc-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha-2-0]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — Requirements for Estimation

All Alpha 2.0 requirements consolidated for estimation. **Risk** is a build/estimate-confidence rating — wider Low→High hour spreads carry more risk. **Size** is a relative t-shirt size. **Low / Avg / High** are 3-point engineering-hour estimates. Notes carry provenance (`FDC` = stated in the Functional Design Criteria; `SE-added` = surfaced during review, not in the FDC) plus rationale. Parent: [[biopredictx-fdc-alpha-2-0]].

## Functional requirements

| ID | Definition | Source | Risk | Comments |
| :-- | :-- | :-- | :-- | :-- |
| FR-A1 | Operations Portal — User Configuration: register new test user; assign Restiv device. | FDC 3.1 | Low | Standard CRUD web UI; no hardware or real-time dependency. |
| FR-A2 | Operations Portal — Device Configuration: set Restiv therapy settings (magnet rotation angles, motor acceleration, rotational frequency). | FDC 3.1 | Medium | Depends on the existing Restiv firmware exposing these parameters over the RPi–Arduino interface; if not exposed, the interface must be defined or extended. |
| FR-A3 | Operations Portal — Device Status: device list; network status. | FDC 3.1 | Low | Conventional status display over device metadata. |
| FR-A4 | Operations Portal — Data Retrieval: wearable raw-data reports; sleep-stage timestamps; therapy reports (start, stop, interrupted). | FDC 3.1 | Low | Reporting over stored data; depends only on the Data Storage Layer being in place. |
| FR-A5 | API & Services Layer — standardized interfaces across UIs, backend, analytics, devices (auth, authorization, device commands, data access, session management). | FDC 3.2 | Medium | Conventional API backbone but must carry real-time device commands and session state; foundational, so defects propagate widely. |
| FR-A6 | Data Acquisition Layer — acquire biometric/telemetry over Bluetooth LE; validation; time-stamping; buffering; streaming (Pi-side ingestion). | FDC 3.2 (DAL) | High | Pi-side of the wearable link; real-time streaming is unproven here and depends entirely on the watch-side path below (FR-A12 / FR-A13). |
| FR-A12 | Garmin on-device data-capture app (Connect IQ) — sample the watch sensors during a session and package measurements for real-time delivery to the Raspberry Pi. | Added (SE) | High | Not in the FDC. Real-time sleep staging needs raw measurements off the watch; a Connect IQ app is the only way to sample sensors on-device during a session. Effort depends on which signals Garmin exposes and at what cadence. `[ASSUMPTION: VivoActive 5 + Connect IQ can expose the needed signals at usable rate — validate with a spike.]` |
| FR-A13 | Wearable→gateway transport — establish the real-time link carrying measurements from the Garmin app to the Raspberry Pi; pairing/onboarding and session time-sync. | Added (SE) | High | Not in the FDC and the dominant unknown for the whole system. Connect IQ apps normally communicate through a paired phone (Connect Mobile); direct watch→Pi BLE is not a standard capability, so a companion/bridge may be required. Garmin Connect cloud sync would violate NFR-A2 timing. Needs a feasibility spike before it can be estimated; if infeasible on Garmin this reopens the FDC wearable-selection item (e.g. Muse EEG). |
| FR-A7 | Analytics Layer — signal filtering; feature extraction; sleep-stage determination; outputs six timestamped stages; COTS/Garmin-first staging. | FDC 3.3 | High | COTS/Garmin staging may not surface emerging-deep-sleep in real time; the FDC open item admits custom software may be required. |
| FR-A8 | Therapy Control Layer — device command transmission; status monitoring; therapy ON/OFF logic. | FDC 3.4 | High | ON logic must satisfy the pre-deep-sleep timing constraint (NFR-A2); the command path depends on the existing firmware accepting live commands at low latency. |
| FR-A9 | Data Storage Layer — persist raw sensor data; processed data; sleep-stage outputs; therapy on/off logs; device status logs. | FDC 3.5 | Low | Conventional tiered storage; hot/warm/cold model already sketched. |
| FR-A10 | Access control — internal vs external user permissions matrix; de-identified User Record and Device Record. | FDC 2 | Low | Standard role-based access; de-identification is lightweight (a de-identified User ID). |
| FR-A11 | Operational state machine — Initialization, Idle, Device Pairing, Ready, Monitoring, Therapy Active, Session Complete, Fault, Recovery; sleep-stage loop repeats 3–6× per session. | FDC 4 | Medium | Well specified but must coordinate real-time transitions; fault-from-any-state and safe therapy shutdown add complexity. |

## Non-functional requirements

| ID | Definition | Source | Risk | Comments |
| :-- | :-- | :-- | :-- | :-- |
| NFR-A1 | Real-time closed-loop operation — system operates as a real-time virtual machine; stream biometrics to detect emerging deep sleep. | FDC Objectives | High | The defining constraint of Alpha 2.0; every time-critical behavior depends on it — including the added Garmin path. |
| NFR-A2 | Therapy timing constraint — therapy must activate before the user enters deep sleep; prioritize an early estimate over a later, higher-confidence one. | FDC 3.4 | High | Hardest requirement in the phase; the FDC flags that COTS staging may fail to meet it and force custom development. |
| NFR-A3 | COTS-first / API-first — use COTS to the greatest extent; minimize custom software. | FDC Objectives | Medium | In direct tension with NFR-A2 and with FR-A12/A13; the Garmin app is itself custom software the COTS-first goal did not anticipate. |
| NFR-A4 | Administrative controls over engineered controls. | FDC Objectives | Low | Deliberately reduces engineering burden; lowers overall build risk. |
| NFR-A5 | Extensible, modular design — build for source/module reuse and a clean Beta transition. | FDC Objectives | Low | Design discipline; minor risk of premature abstraction. |
| NFR-A6 | De-identification — maintain a unique de-identified User ID per participant. | FDC System Assets | Low | Lightweight compared with the old scope's engineered privacy stack. |
| NFR-A7 | Fault tolerance / safe state — Fault reachable from any state; preserve data; safe state; disable therapy; then Recovery. | FDC 4 | Medium | Safety-relevant; disabling therapy cleanly on any fault needs care but is bounded. |
| NFR-A8 | [Beta] Single frontend codebase deployable as web now and iOS/Android later. | FDC 14 | Low | Forward-looking groundwork; not Alpha-critical. |
| NFR-A9 | [Beta] Clearly defined interface point for a future proprietary black-box algorithm. | FDC 14 | Medium | Must define the right seam now without the algorithm existing; risk the interface guesses wrong. |

## Estimation note

The three High-risk items on the critical path — FR-A6, FR-A12, FR-A13 (the Garmin→Pi data path) — cannot be reliably estimated until a time-boxed feasibility spike proves a real-time watch→Pi path exists on the VivoActive 5. If it does not, the FDC's wearable-selection open item reopens (e.g. Muse EEG), which changes the shape of the build. Recommend scoping that spike as an explicit first task before committing numbers.
