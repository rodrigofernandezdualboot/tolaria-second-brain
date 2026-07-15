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

Hours are 3-point engineering estimates (Low / Avg / High), matching the estimation note [[biopredictx-alpha2-requirements-estimation]].

| ID | Requirement (short) | Risk | Size | Low | Avg | High |
| :-- | :-- | :-- | :-: | --: | --: | --: |
| FR-A1 | Ops Portal — user configuration | Low | S | 11 | 13 | 14 |
| FR-A2 | Ops Portal — device configuration (therapy params) | Low | S | 10 | 12 | 13 |
| FR-A3 | Ops Portal — device status | Low | S | 8 | 9 | 10 |
| FR-A4 | Ops Portal — data retrieval / reports | Low | M | 20 | 23 | 26 |
| FR-A5 | API & Services Layer | Low | M | 20 | 23 | 26 |
| FR-A6 | Data Acquisition Layer — real-time BLE ingest (Pi-side) | High | M? | 26 | 52 | 78 |
| FR-A7 | Analytics Layer — sleep-stage determination (COTS/Garmin-first) | Low | XL? | 65 | 75 | 85 |
| FR-A8 | Therapy Control Layer — command tx, status, ON/OFF logic | High | M | 16 | 32 | 48 |
| FR-A9 | Data Storage Layer | Low | S | 10 | 12 | 13 |
| FR-A10 | Access control — permissions matrix, light de-id | Low | XS | 3 | 3 | 4 |
| FR-A11 | Operational state machine | Medium | M | 16 | 24 | 32 |
| Infrastructure | Environments, CI/CD, Pi + device provisioning | Low | S | 5 | 6 | 7 |

### SE-added functional requirements (from derisking — not in the FDC)

| ID | Requirement (short) | Risk | Size | Low | Avg | High |
| :-- | :-- | :-- | :-: | --: | --: | --: |
| FR-A12 | Garmin on-device capture app (Connect IQ) | Medium | L | 32 | 48 | 64 |
| FR-A13 | Watch→Pi real-time transport | Medium | L | 32 | 48 | 64 |

**Alpha 2.0 total (functional + infrastructure): 274 / 379 / 484 engineering-hours (Low / Avg / High).**

---

## Read

The two scopes size very differently. **Scope v2 is front-loaded with XL/L items** — ingestion, de-identification ETL, and the SageMaker platform — because enterprise compliance and 500K-device scale make even routine capabilities heavy. **Alpha 2.0 is mostly S/M** (an internal portal over conventional storage and access control), totalling **274 / 379 / 484 engineering-hours** (Low / Avg / High).

After the refined estimate, the range is driven by two things: the sheer size of **FR-A7** (65–85h, the largest single line, but now *low*-variance — treated as a well-scoped build), and the wide spreads on the two remaining High-risk items, **FR-A6** (26–78h) and **FR-A8** (16–48h) — real-time ingestion and therapy control. The Garmin path (FR-A12/A13) settled to Medium/L after the derisking. The open feasibility question is no longer "can we build FR-A7" but the timing constraint it serves (NFR-A1/A2) — whether therapy can fire before deep-sleep onset — which is what the Phase 1 gate exists to answer.
