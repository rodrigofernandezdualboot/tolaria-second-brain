---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-scope-v2]]"
  - "[[biopredictx-fdc-alpha-2-0]]"
status: Active
---
# BioPredictX Scope Requirements — Validation & Comparison

## Old Scope — Scope v2 \(production cloud platform\)

The scope doc cites requirement codes from the underlying FDRD. Extracted exactly as cited; sub-ranges \(e.g. "FR-01.1–01.5"\) are kept as the doc groups them rather than split into invented sub-items.

### Functional requirements

| ID | Definition \(as stated\) |
| --- | --- |
| FR-01.2 | Secure device authentication — X.509 device certs from a private CA; AWS IoT Core device registry operational. |
| FR-01.1–01.5 | Nightly telemetry ingestion pipeline — MQTT command channel + HTTPS token vending for Protobuf payloads \>128KB; SQS buffer; MQTT acknowledgement only on confirmed S3 persistence. |
| FR-02 | Strict de-identification — Glue pipeline extracts PII on ingest into an isolated DynamoDB vault; clinical records linked only via one-way hash pseudonym. |
| FR-02.1–02.4 | De-identification ETL operational — Glue extracting PII, DynamoDB PII vault active, pseudonym hash linking clinical records, Comprehend Medical PHI scrubbing. |
| FR-02.4 | Amazon Comprehend Medical — NLP-based PHI scrubbing of free-text fields during ETL. |
| FR-02.5 | Crypto-shredding — raw landing zone to Glacier after 7 days; KMS CMK deletion on Right-to-be-Forgotten enables GDPR compliance without physical deletion. |
| FR-03.1 | Curated clinical data lake — S3 Tables \(Parquet\), partitioned by date and cohort\_id; raw landing-zone Glacier lifecycle. |
| FR-03.2–03.4 | SageMaker ML environment — private VPC, PrivateLink-only access, Feature Store, model registry with data-lineage tracking. |
| FR-03.4 | SaMD traceability — every ML model artifact linked to the specific versioned dataset used in training. |
| FR-04 | Mobile app is a consumer of the AppSync GraphQL API — not a data-pipeline participant \(cloud is system of record\). |
| FR-04.1–04.2 | AppSync GraphQL API — Cognito user-pool auth, patient identity resolved against the PII vault, summary health statistics, trend history. |
| FR-04.3 | Real-time risk-alert delivery via WebSocket subscriptions. |
| FR-05.1 | OTA firmware updates via AWS IoT Jobs \(digitally signed\) — acknowledged as a platform capability, **explicitly excluded** from the Dualboot engagement. |

### Non-functional requirements

| ID | Definition \(as stated\) |
| --- | --- |
| NFR-01.1–01.4 | Security/compliance baseline — CloudTrail logging, KMS CMKs, TLS 1.2+ \(group range as cited\). |
| NFR-01.2 | Encryption — all data at rest via KMS Customer Managed Keys; all data in transit via TLS 1.2+. |
| NFR-01.3 | Network isolation — all database and ML access exclusively via VPC endpoints \(PrivateLink\); no public IPs in the intelligence layer. |
| NFR-01.4 | 6-year audit logging — all API access, data read/write, and config changes logged to CloudTrail with 6-year minimum retention. |
| NFR-02.1 | Thundering-herd ingestion — 500,000 devices uploading 1MB Protobuf payloads within a 4-hour nightly window. |
| NFR-02.2–02.3 | API performance & data freshness — AppSync p95 \<200ms; data fresh within 4 hours of upload completion. |
| NFR-03 / NFR-03.1 | Availability & DR — RPO 24h, RTO 4h via cross-region replication. |
| NFR-03.2 | Uptime — 99.9% for API and ingestion layers during business hours. |

*Note: NFR-01.1 is named only inside the "NFR-01.1–01.4" range; the doc does not individually define it, so it is not given a standalone definition here.*

---

## New Scope — FDC Alpha 2.0 \(constrained clinical feasibility\)

The FDC has no FR/NFR numbering. IDs below \(FR-A\#, NFR-A\#\) are **my labels for functions and criteria the document already states** — not new requirements. Each traces to a named section/layer.

### Functional requirements

| ID | Definition \(as stated\) | Source section |
| --- | --- | --- |
| FR-A1 | Operations Portal — User Configuration: register new test user; assign Restiv device. | 3.1 |
| FR-A2 | Operations Portal — Device Configuration: set Restiv therapy settings \(magnet rotation angles, motor acceleration, rotational frequency\). | 3.1 |
| FR-A3 | Operations Portal — Device Status: device list; network status. | 3.1 |
| FR-A4 | Operations Portal — Data Retrieval: wearable raw-data reports; sleep-stage timestamps; therapy reports \(time start, time stop, therapy interrupted\). | 3.1 |
| FR-A5 | API & Services Layer — standardized interfaces across UIs, backend, analytics, and devices \(auth, authorization, device commands, data access, session management\). | 3.2 |
| FR-A6 | Data Acquisition Layer — acquire biometric/telemetry over Bluetooth LE; data validation & time-stamping; buffering & streaming. | 3.2 \(DAL\) |
| FR-A7 | Analytics Layer — signal filtering, feature extraction, sleep-stage determination; outputs timestamped stages \(Awake, Light Sleep, Emerging Deep Sleep, Deep Sleep, REM, Unknown\); staging via open-source/COTS, Garmin SDK/Connect first. | 3.3 |
| FR-A8 | Therapy Control Layer — device command transmission, status monitoring, therapy ON/OFF logic; input timestamped sleep stages, output therapy ON/OFF. | 3.4 |
| FR-A9 | Data Storage Layer — persist raw sensor data, processed data, sleep-stage outputs, therapy on/off logs, device status logs. | 3.5 |
| FR-A10 | Access control — internal \(Administrator/Operator\) vs external \(Test User\) permissions matrix; de-identified User Record and Device Record. | 2, 2a, 2b |
| FR-A11 | Operational state machine — Initialization, Idle, Device Pairing, Ready, Monitoring, Therapy Active, Session Complete, Fault, Recovery, with defined functions and exit conditions; sleep-stage loop repeats 3–6× per session. | 4 |

### Non-functional requirements

| ID | Definition \(as stated\) | Source section |
| --- | --- | --- |
| NFR-A1 | Real-time closed-loop operation — system operates as a real-time virtual machine; stream biometrics to detect emerging deep sleep. | Product Overview / Objectives |
| NFR-A2 | Therapy timing constraint — therapy **must activate before the user enters deep sleep**; prioritize an early estimated indication over a later, higher-confidence one. | 3.4 |
| NFR-A3 | COTS-first / API-first — identify and use COTS technologies; minimize all custom software development. | Objectives |
| NFR-A4 | Administrative controls over engineered controls. | Objectives |
| NFR-A5 | Extensible, modular design — build for source-code/module reuse and seamless evolution into Beta. | Objectives |
| NFR-A6 | De-identification — maintain a unique de-identified User ID per test participant. | System Assets |
| NFR-A7 | Fault tolerance / safe state — Fault reachable from any state; preserve collected data, place system in safe state, disable therapy if required, then Recovery. | 4 |
| NFR-A8 | *[Beta / forward-looking]* Single frontend codebase deployable as web now and iOS/Android later. | 14 |
| NFR-A9 | *[Beta / forward-looking]* Clearly defined interface point for a future proprietary "black box" predictive algorithm to plug in. | 14 |

### FDC open items \(NOT requirements — assumptions/risks\)

- **Wearable selection** — risk of change. Muse EEG has all needed sensors in one device; Garmin lacks the EEG needed for continued deep-sleep testing. Assumption: Alpha 2.0 integrates with Garmin software; Muse EEG may be used external to the system.
- **Therapy control logic** — assumption that "on"-signal logic can be derived from simple alterations to standard COTS sleep staging; risk it fails to time therapy *prior* to deep-sleep onset, forcing custom software this phase.

---

## Comparison

### By dimension

| Dimension | Old Scope \(v2\) | New Scope \(FDC Alpha 2.0\) |
| --- | --- | --- |
| Goal | Commercial D2C platform, 500K-unit scale, FDA Class 2 pathway | Constrained clinical feasibility test of the Restiv prototype |
| Data flow | Device → Cloud → App; nightly batch upload | Device ↔ local gateway ↔ cloud; real-time streaming |
| Trigger model | Retrospective analysis to adjust future protocols | Real-time therapy activation before deep-sleep onset \(NFR-A2\) |
| User interface | Patient-facing iOS/Android \(GraphQL/Cognito\) | Internal Operations Portal \(Web UI\); consumer app out of scope |
| Algorithm | Proprietary ML in SageMaker + Feature Store | COTS/open-source staging, Garmin-first; proprietary deferred to Beta |
| Edge hardware | Abstracted under-bed device \(firmware out of scope\) | Raspberry Pi 4 + Arduino microcontroller + Garmin + Muse EEG |
| Compliance posture | Engineered: Glue de-id, DynamoDB PII vault, crypto-shredding, 6-yr CloudTrail, PrivateLink, CMKs, RPO/RTO | Administrative: de-identified User ID + manual process \("admin over engineered controls"\) |
| Scale/performance | 500K devices, 1MB payloads, 4-hr window, p95 \<200ms, 99.9% uptime | Minimal-use Alpha test regime; no scale targets stated |

### Requirement-theme mapping \(old → new\)

| Theme | Old scope | New scope | What changed |
| --- | --- | --- | --- |
| Ingestion | FR-01.1–01.5 \(MQTT/HTTPS batch\) | FR-A6 \(BLE real-time\) + NFR-A1 | Batch cloud ingest → real-time local acquisition |
| Device auth | FR-01.2 \(X.509/IoT Core\) | — | No equivalent stated; local pairing \(FR-A11\) instead |
| De-identification | FR-02, 02.1–02.4, 02.5 \(Glue, vault, Comprehend, crypto-shred\) | NFR-A6 \(de-identified User ID\) | Heavy engineered privacy → lightweight de-id + admin controls \(NFR-A4\) |
| Data lake / ML | FR-03.1–03.4 \(SageMaker, Feature Store, SaMD lineage\) | FR-A7 \(COTS staging\) + FR-A9 \(storage\) | Proprietary ML → COTS/Garmin; proprietary deferred \(NFR-A9\) |
| App / engagement | FR-04, 04.1–04.3 \(AppSync, Cognito, WebSocket\) | FR-A1–A4 \(Ops Portal\) | Patient app → internal admin portal; consumer app out of scope |
| Therapy control | — \(not a scope-v2 requirement\) | FR-A8 + NFR-A2 | New: real-time closed-loop therapy with pre-deep-sleep timing |
| Fleet / OTA | FR-05.1 \(IoT Jobs, excluded\) | — | Dropped in Alpha |
| Security/compliance | NFR-01.2–01.4 \(CMK, PrivateLink, 6-yr audit\) | NFR-A4, NFR-A6 | Engineered compliance → administrative controls |
| Performance/scale | NFR-02.1–02.3 \(500K, p95 \<200ms\) | — | No scale targets; minimal-use regime |
| Availability/DR | NFR-03/03.1/03.2 \(RPO/RTO, 99.9%\) | NFR-A7 \(fault/safe-state\) | Enterprise DR → basic fault tolerance |
| Extensibility | — | NFR-A5, NFR-A8, NFR-A9 | New: modular groundwork for Beta |
