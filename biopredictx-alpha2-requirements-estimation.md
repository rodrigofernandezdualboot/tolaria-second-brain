---
type: Note
belongs_to: "[[biopredictx-fdc-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha-2-0]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — Requirements for Estimation

All Alpha 2.0 requirements consolidated for estimation. **Risk** is a build/estimate-confidence rating — wider Low→High hour spreads carry more risk. **Size** is a relative t-shirt size. **Low / Avg / High** are 3-point estimates in **person-days** (working days of effort). Notes carry provenance (`FDC` = stated in the Functional Design Criteria; `SE-added` = surfaced during review, not in the FDC) plus rationale. Parent: [[biopredictx-fdc-alpha-2-0]].

## Functional requirements

| ID | Definition | Risk | Size | Low | Avg | High | Notes |
| :-- | :-- | :-- | :-- | --: | --: | --: | :-- |
| FR-A1 | Operations Portal — User Configuration: register new test user; assign Restiv device. | Low | S | 11 | 13 | 14 | FDC 3.1. Standard CRUD web UI. |
| FR-A2 | Operations Portal — Device Configuration: set Restiv therapy settings (magnet rotation angles, motor acceleration, rotational frequency). | Low | S | 10 | 12 | 13 | FDC 3.1. UI + parameter plumbing to the device. |
| FR-A3 | Operations Portal — Device Status: device list; network status. | Low | S | 8 | 9 | 10 | FDC 3.1. Conventional status display. |
| FR-A4 | Operations Portal — Data Retrieval: wearable raw-data reports; sleep-stage timestamps; therapy reports (start, stop, interrupted). | Low | M | 20 | 23 | 26 | FDC 3.1. Reporting over stored data. |
| FR-A5 | API & Services Layer — standardized interfaces across UIs, backend, analytics, devices (auth, authorization, device commands, data access, session management). | Low | M | 20 | 23 | 26 | FDC 3.2. Conventional API backbone. |
| FR-A6 | Data Acquisition Layer — acquire biometric/telemetry over Bluetooth LE; validation; time-stamping; buffering; streaming (Pi-side ingestion). | High | M? | 26 | 52 | 78 | FDC 3.2. Pi-side of the wearable link; wide spread reflects real-time streaming uncertainty and dependence on FR-A12/A13. |
| FR-A7 | Analytics Layer — signal filtering; feature extraction; sleep-stage determination; outputs six timestamped stages; COTS/Garmin-first staging. | Low | XL? | 65 | 75 | 85 | FDC 3.3. Largest single item but well-scoped (narrow spread). The hard part it depends on — timing therapy before deep-sleep onset — is tracked under NFR-A1/A2, not here. |
| FR-A8 | Therapy Control Layer — device command transmission; status monitoring; therapy ON/OFF logic. | High | M | 16 | 32 | 48 | FDC 3.4. Command path + ON/OFF logic; spread reflects firmware-interface and timing uncertainty. |
| FR-A9 | Data Storage Layer — persist raw sensor data; processed data; sleep-stage outputs; therapy on/off logs; device status logs. | Low | S | 10 | 12 | 13 | FDC 3.5. Conventional tiered storage. |
| FR-A10 | Access control — internal vs external user permissions matrix; de-identified User Record and Device Record. | Low | XS | 3 | 3 | 4 | FDC 2. Light RBAC + de-identified IDs. |
| FR-A11 | Operational state machine — Initialization, Idle, Device Pairing, Ready, Monitoring, Therapy Active, Session Complete, Fault, Recovery; sleep-stage loop repeats 3–6× per session. | Medium | M | 16 | 24 | 32 | FDC 4. Coordinates real-time transitions; fault-from-any-state and safe therapy shutdown. |
| FR-A12 | Garmin on-device data-capture app (Connect IQ) — sample the watch sensors during a session and package measurements for real-time delivery to the Raspberry Pi. | Medium | L | 32 | 48 | 64 | SE-added (not in FDC). Connect IQ capture app; foreground-only, limited signals; sized after derisking. |
| FR-A13 | Wearable→gateway transport — establish the real-time link carrying measurements from the Garmin app to the Raspberry Pi; pairing/onboarding and session time-sync. | Medium | L | 32 | 48 | 64 | SE-added (not in FDC). BLE (Pi-as-peripheral) or Wi‑Fi HTTP path — both viable per derisking; bench spike still recommended. |
| Infrastructure | Environments, CI/CD, Raspberry Pi + device provisioning, developer tooling. | Low | S | 5 | 6 | 7 | Cross-cutting setup not tied to a single FR. |
| **Total** | | | | **274** | **379** | **484** | Person-days (3-point engineering estimate). |

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

Total engineering estimate: **274 / 379 / 484 person-days** (Low / Avg / High) — about **$186K / $258K / $329K** engineering-only at $680/day ($85/hr × 8h). Loaded with PM/QA/design (+25%) and contingency (+10%), the all-in lands near **$355K** in the average case; the full per-phase breakdown is in [[biopredictx-alpha2-phased-proposal]].

Risk now sits with the two widest-spread items, **FR-A6** (real-time ingestion, 26–78 days) and **FR-A8** (therapy control, 16–48 days), both High. FR-A7 is the largest single line (65–85 days) but low-variance — a big but predictable build; the genuinely hard part it depends on, timing therapy before deep-sleep onset, lives in **NFR-A1/A2** and remains the open question the Phase 1 gate must answer. The Garmin path (FR-A12/A13) is now Medium after the derisking, which found viable BLE and Wi‑Fi transport options.
