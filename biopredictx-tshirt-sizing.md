---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-scope-requirements-comparison]]"
  - "[[biopredictx-alpha2-requirements-estimation]]"
status: Active
_organized: true
---

# BioPredictX — Functional Requirements T-Shirt Sizing

Relative effort sizing of the functional requirements for both scopes. Source lists: [[biopredictx-scope-requirements-comparison]]. This is a **relative complexity exercise, not a commitment** — no hours attached, and several Alpha 2.0 sizes are contingent on the derisking spikes.

**Legend (relative complexity for a blended senior team):**

- **XS** — trivial; a few days.
- **S** — ~1–2 weeks; single engineer, well-understood.
- **M** — ~2–4 weeks; some integration or design work.
- **L** — ~1–2 months; multi-component or security/compliance-critical.
- **XL** — 2+ months; major subsystem or high uncertainty.
- **?** — size contingent on an unresolved feasibility question; could move a full size or more.

---

## Original version (Scope v2 — production cloud platform)

| ID | Requirement (short) | Size | Rationale |
| :-- | :-- | :-: | :-- |
| FR-01.2 | Secure device auth — X.509 private CA + IoT Core registry | L | PKI + device registry; security-critical, fiddly to get right. |
| FR-01.1–01.5 | Nightly ingestion pipeline — MQTT + HTTPS token vending, Protobuf, SQS, ack-on-persist | XL | Core high-throughput pipeline built for the thundering-herd scale. |
| FR-02 | Strict de-identification (umbrella design) | L | De-id/pseudonymization strategy; overlaps the ETL row below. |
| FR-02.1–02.4 | De-identification ETL — Glue, DynamoDB PII vault, hashing, Comprehend | XL | Compliance-critical, most complex data-plane work in the platform. |
| FR-02.4 | Amazon Comprehend Medical — PHI scrubbing | M | NLP integration; subset of the ETL above. |
| FR-02.5 | Crypto-shredding — Glacier lifecycle + KMS CMK deletion for RTBF | M | Lifecycle + key-management workflow. |
| FR-03.1 | Curated clinical data lake — S3 Tables/Parquet, partitioning | L | Lake design, partitioning, lifecycle at scale. |
| FR-03.2–03.4 | SageMaker ML environment — private VPC, PrivateLink, Feature Store, registry | XL | Full isolated ML platform stand-up. |
| FR-03.4 | SaMD traceability — model↔dataset lineage | L | Regulatory governance overhead; subset of the ML env but heavy. |
| FR-04 | App as consumer of AppSync API (architecture) | S | Mostly a design/contract decision; the app itself is client-owned. |
| FR-04.1–04.2 | AppSync GraphQL API — Cognito auth, PII-vault resolution, summary stats/trends | L | Schema, auth, identity resolution, query surface. |
| FR-04.3 | Real-time risk alerts via WebSocket | M | Subscription push layer. |
| FR-05.1 | OTA via AWS IoT Jobs | — | Explicitly excluded from the engagement; not sized. |

*Overlap caveat: the FR-02 umbrella and FR-02.1–02.4, and FR-03.2–03.4 vs. FR-03.4, describe the same work at different granularity — do not sum their sizes.*

---

## Alpha 2.0 (FDC — constrained clinical feasibility)

| ID | Requirement (short) | Size | Rationale |
| :-- | :-- | :-: | :-- |
| FR-A1 | Ops Portal — user configuration | S | Standard CRUD web UI. |
| FR-A2 | Ops Portal — device configuration (therapy params) | M | UI plus parameter plumbing to the device; depends on the firmware interface. |
| FR-A3 | Ops Portal — device status | S | Conventional status display. |
| FR-A4 | Ops Portal — data retrieval / reports | S | Reporting over stored data. |
| FR-A5 | API & Services Layer | M | Backbone: auth, device commands, session management. |
| FR-A6 | Data Acquisition Layer — real-time BLE ingest (Pi-side) | L ? | Real-time streaming + validation + time-sync; size hinges on the transport spike (FR-A13). |
| FR-A7 | Analytics Layer — real-time sleep staging / pre-onset detection | XL ? | The hard one. Real-time staging isn't COTS; "before deep sleep" is a forecasting problem likely needing custom EEG work. Could grow if the requirement isn't relaxed. |
| FR-A8 | Therapy Control Layer — command tx, status, ON/OFF logic | L | Carries the pre-deep-sleep timing and the firmware command-path dependency. |
| FR-A9 | Data Storage Layer | S | Conventional tiered storage. |
| FR-A10 | Access control — permissions matrix, light de-id | S | Standard RBAC; de-id is a single de-identified User ID. |
| FR-A11 | Operational state machine | M | Real-time transitions, fault-from-any-state, safe therapy shutdown. |

### SE-added functional requirements (from derisking — not in the FDC)

| ID | Requirement (short) | Size | Rationale |
| :-- | :-- | :-: | :-- |
| FR-A12 | Garmin on-device capture app (Connect IQ) | L ? | New custom watch app; foreground-only capture, limited signals; size confirmed only after a capability spike. |
| FR-A13 | Watch→Pi real-time transport | L ? | BLE (Pi-as-peripheral) or Wi‑Fi HTTP path, both unproven; jumps to XL if infeasible on Garmin and the wearable must change (Muse EEG). |

---

## Read

The two scopes size very differently. **Scope v2 is front-loaded with XL/L items** — ingestion, de-identification ETL, and the SageMaker platform — because enterprise compliance and 500K-device scale make even routine capabilities heavy. **Alpha 2.0 is mostly S/M** (an internal portal over conventional storage and access control), with the effort concentrated in a small cluster of hard, uncertain items: **FR-A6, FR-A7, FR-A8, and the added FR-A12/A13** — the real-time sensing-to-therapy loop.

The practical implication for estimating: Alpha 2.0's total is dominated by four-to-five requirements whose sizes carry a `?`. Until the feasibility spikes resolve (real-time staging timing, and watch→Pi transport), the Alpha 2.0 number has a wide error bar — the S/M majority is predictable, but the XL/L cluster is where the range lives. If NFR-A2 ("before deep sleep") holds and Garmin can't deliver real-time stages, FR-A7 plus a wearable switch could dominate the whole estimate.
