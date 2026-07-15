---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
status: Superseded
_organized: true
---

# BioPredictX Scope v2 (Old Scope)

Dualboot Partners scope, staffing, and pricing definition for the BioPredictX Longitudinal Medical IoT Platform. February 2026, v2. This is the **prior production-focused scope** that Alpha 2.0 pivots away from. See [[biopredictx-fdc-alpha-2-0]] for the new approach and [[biopredictx-alpha-2-0]] for the side-by-side.

## Context & objective

Connected sleep-therapy device using static magnetic fields to increase delta-wave (deep) sleep. The software problem is the platform behind the device: a HIPAA/GDPR-compliant, cloud-native system ingesting nightly telemetry from up to 500,000 deployed units, protecting patient identity, surfacing clinical insight, and supporting a future FDA Class 2 (SaMD) pathway.

## Architecture

Critical data-flow decision: **Device → Cloud → App**, not Device → App → Cloud. The cloud is the system of record; the mobile app is only a consumer of the AppSync GraphQL API, not a pipeline participant. Framed as a compliance and scalability requirement.

Four layers:

- **Edge** — physical hardware and firmware. Out of scope for Dualboot.
- **Ingestion** — hybrid MQTT (command/control) + HTTPS token vending with S3 presigned URLs for payloads >128KB. X.509 device certs from a private CA, Protobuf serialization, SQS burst buffering. MQTT ack only after confirmed S3 persistence.
- **Intelligence** — AWS Glue event-driven ETL for immediate de-identification → S3 Tables (Parquet, partitioned by date and cohort_id) → isolated SageMaker ML environment in a private VPC (PrivateLink only, no public IPs) → Feature Store → model registry with full SaMD data lineage.
- **Engagement** — AppSync GraphQL API, Cognito user pools, identity resolved against the PII vault at query time, WebSocket push for real-time risk alerts. p95 API target <200ms.

## Non-negotiable compliance

Strict de-identification (Glue extracts PII on ingest into an isolated DynamoDB vault; clinical records linked only by one-way hash). Crypto-shredding (raw data to Glacier after 7 days; KMS CMK deletion satisfies GDPR right-to-be-forgotten). Amazon Comprehend Medical for PHI scrubbing of free text. 6-year CloudTrail audit retention. Network isolation via VPC endpoints. KMS CMK encryption at rest, TLS 1.2+ in transit. SaMD traceability linking every model artifact to its versioned training dataset. RPO 24h / RTO 4h via cross-region replication; 99.9% uptime target.

## Team & pricing

Lean five-person senior team, blended $85/hr: Product Manager, Cloud/Security Architect, Backend/Full-Stack Engineer, Data/AI Engineer, QA/Compliance Engineer. One full-stack engineer (not two), with the Architect and Data Engineer carrying real build hours.

| Phase | Duration | Hours | Cost | % |
| :-- | :-- | :-- | :-- | :-- |
| 1 — Strategy & Architecture | 4–5 wks | 420 | $35,700 | 21% |
| 2 — MVP Build (Device→Cloud→App) | 8–10 wks | 1,080 | $91,800 | 53% |
| 3 — QA, Compliance & Hardening | 4–5 wks | 540 | $45,900 | 26% |
| **Total** | **16–20 wks** | **2,040** | **$173,400** | **100%** |

$15,000 deposit on SOW effective date, credited to final invoice. Monthly invoices, AWS/third-party passed through at cost.

## vs. prior Gemini analysis

Total $173,400 vs. Gemini's $178,500–$219,300 — tighter scope, no redundancy. 3 phases aligned to the SOW template vs. Gemini's 4. Phase 2 priced fuller at $91,800 (two full months at capacity) vs. Gemini's ~$47,600–$51,000. Data flow corrected to cloud-first. OTA/fleet management (FR-05.1) acknowledged but explicitly excluded.

## Explicit exclusions

Embedded firmware, hardware OTA implementation, physical device work; mobile app UI (client-owned); production ML tuning and clinical validation; FDA 510(k) prep; end-user support; AWS cost optimization / MAP-SBAI funding; third-party EHR/HL7-FHIR integration beyond basic API design.

## Key risks

Thundering-herd ingestion (500K devices, 1MB payloads, 4-hour window). De-PII complexity (NLP for free-text PHI, crypto-shredding). OTA firmware risk (out of scope). Regulatory dual-track (GDPR + FDA Class 2 simultaneously). ML scope creep (Phase 2 delivers the environment, not production models).

*[ASSUMPTION] Filed as "old scope" per Rodrigo's framing; source doc is `biopredictx_scope_v2.docx`, MSA reference Aug 13, 2025.*
