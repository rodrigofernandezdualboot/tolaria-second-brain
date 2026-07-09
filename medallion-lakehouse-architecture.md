---
type: Note
related_to: "[[software-architecture]]"
url: https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion
_organized: true
---

# Medallion Lakehouse Architecture

The medallion architecture is a data design pattern that organizes data into a series of layers denoting increasing data quality as it flows through a lakehouse. It progressively improves the structure and reliability of data from **Bronze ⇒ Silver ⇒ Gold**, making it more suitable for BI and machine learning. It's also called a *multi-hop architecture*, and is a recommended best practice rather than a requirement.

The pattern guarantees atomicity, consistency, isolation, and durability (ACID) as data passes through validations and transformations before landing in a layout optimized for analytics.

## The three layers

| | Bronze (raw) | Silver (validated) | Gold (enriched) |
|---|---|---|---|
| **What happens** | Raw data ingestion | Data cleaning and validation | Dimensional modeling and aggregation |
| **Primary users** | Data engineers, data operations, compliance/audit | Data engineers, data analysts, data scientists | Business/BI analysts, data scientists & ML engineers, executives, operational teams |

### Bronze — ingest raw data

- Maintains the raw state of the source in its original format; single source of truth preserving fidelity.
- Appended incrementally, grows over time, and retains all historical data to enable reprocessing and auditing.
- Feeds silver workloads — not intended for direct analyst/data-scientist access.
- Handles any mix of streaming and batch from cloud object storage (S3, GCS, ADLS), message buses (Kafka, Kinesis), and federated systems.
- **Minimal validation.** Store most fields as string, VARIANT, or binary to guard against unexpected schema changes; add metadata columns (e.g. `_metadata.file_name`).

### Silver — validate and deduplicate

- Built by reading from one or more bronze/silver tables and writing to silver tables (avoid writing directly from ingestion — schema changes and corrupt records cause failures). Prefer streaming reads from append-only bronze sources; reserve batch reads for small dimensional tables.
- Always includes at least one validated, non-aggregated representation of each record.
- Operations performed here: schema enforcement, null/missing-value handling, deduplication, out-of-order & late-arriving data resolution, quality checks, schema evolution, type casting, and joins.
- Data modeling often begins here — representing nested/semi-structured data via VARIANT, JSON strings, structs/maps/arrays, or by flattening/normalizing into multiple tables.

### Gold — power analytics

- Highly refined, aggregated views that drive dashboards, ML, applications, and reporting.
- Aligns with business logic and requirements; optimized for query and dashboard performance.
- Modeled dimensionally (relationships and measures) so analysts can answer domain-specific questions.
- Teams often build multiple gold layers per business domain (e.g. HR, finance, IT) and create aggregate measures (averages, counts, min/max) tailored to reporting needs.

## Example (business operations, `ops` catalog)

- **Bronze (`ops.bronze`)** — ingests raw data from cloud storage, Kafka, and Salesforce; no cleanup.
- **Silver (`ops.silver`)** — cleans and joins data: `customer_transactions`, `account_opportunities`, `leads_cleaned`.
- **Gold (`ops.gold`)** — business-facing datasets: `customer_spending`, `account_performance`, `sales_pipeline_summary`, `business_summary`.

---

Source: [Microsoft Learn — What is the medallion lakehouse architecture?](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion)
