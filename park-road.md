---
type: Project
status: Active
_organized: true
---

# Park Road

Park Road Partners LP (PE, 1230 6th Avenue, NY) — evaluating an investment in **Granite** (Gulfstream Outsourcing and Specialized Billing, LLC), an outsourced revenue cycle management / billing provider for healthcare providers and hospital systems, focused on **accident-related claims (ARC)**.

Primary source: *EY-Parthenon "Project Granite" Product, Technology & Cybersecurity Due Diligence — Summary of Findings, May 12, 2026* (working draft, 50 pp). Engagement lead: Ryan Jacobson (Partner), Peter Zadrozny (MD). Client contact: Matt Dubbioso. ~2 weeks of diligence, 5 hours of management meetings incl. a 1-hr platform demo.

**Dualboot's angle:** platform takeover as delivery partner post-close. Roadmap and per-task risk register in [[park-road-granite-platform-takeover-roadmap]].

## Investment thesis (EY-P's understanding)

- ProWeb, Granite's proprietary platform, is a credible and scalable foundation for value creation.
- Granite is under-institutionalized — limited process and data maturity — creating professionalization levers.
- Margin expansion via optimizing the offshore services model and layering AI automation across workflows (e.g. payer identification), moving toward software-like economics.

## Business shape

- Revenue ~$4.8m (2023) → ~$5.9m (2024) → ~$9.5m (2025); gross profit ~$2.5m → ~$3.6m → ~$6.5m.
- Four solutions by lifecycle entry point: Day 1 full-service ARC RCM (42% of volume, 8–15% of collections), Aged secondary review (32%, 20–30%), Historical underpayment recovery (25%, 25–30%), Discovery coverage detection (2%, $12–25/claim).
- ~100 customers, ~1,000 service delivery locations, ~1m PHI records in ProWeb.
- ~52% of workflow steps automated. Remaining cost is concentrated in manual, repeat follow-up loops (payer/patient/employer outreach, coverage discovery, exception handling).

## Technology findings — ProWeb

**Architecture (p.23).** Built almost entirely around T-SQL stored procedures and triggers; effectively no business logic in the application layer. The app server is a passthrough over an ADO-based DB connector. Web API + WCF provide integration pathways, WCF being a legacy pattern. Performant at current scale, but concentrating logic in the database makes workflows hard to test, maintain, and extend — and creates friction for AI that needs to *drive* actions inside ProWeb rather than write results back adjacent to it.

**Stack (p.24).** UI: Bootstrap 3.x/4.x, jQuery 3.x, Knockout.js 3.x, KendoUI, Razor. App: C# / .NET Framework 4.5 (EOL since 2022) + Python. DB: SQL Server 2017 `[ASSUMPTION — version not confirmed; inferred by EY-P from management's stated 2027 end-of-support date]`. Integration: REST API, IIS-hosted XML webhook, WCF, SFTP. Data tooling: Power BI. Near-term stabilization step identified by management is .NET 4.8.1 — not full modernization to .NET Core. React is under evaluation for the frontend but is not a confirmed roadmap decision.

**Verified end-of-support dates** (Microsoft Learn, checked 29 Jul 2026): SQL Server 2017 extended support ends **12 Oct 2027**; Windows Server 2016 extended support ends **12 Jan 2027**. Against EY-P's 22–28 month tech debt timeline, both platform floors expire mid-programme. EY-P rates the OS upgrade Low priority and leaves it unsized.

**Technical debt (pp.25–26).** Ranked by criticality: .NET modernization (M), business logic refactoring out of stored procedures (**XL** — the biggest item), frontend/UI modernization (L), infrastructure/DB modernization (M), TFS → Azure DevOps (S), CI/CD pipeline (S), Windows Server 2016 upgrade (unsized, IT-led), data warehouse (unsized — not feasible near-to-medium term).

Cost to remediate: **~$305k–$405k** tech debt + **~$30k–$50k** platform/SDLC = ~$335k–$455k; ~**$365k–$485k** including the modernization-dependent platform enhancements. Timeline ~22–28 months. Sequencing insight: automation tied to ProWeb business logic (claim state, Next Actions, payer routing, coverage logic) is gated on .NET and stored-procedure remediation; automation *outside* ProWeb (Python workflows calling BERT/XGBoost) can scale now. Architectural extensibility is **not** explicitly sized as a standalone initiative — only partially covered by business logic refactoring.

**Open source (p.27).** 10 vulnerabilities. Two iTextSharp are the live exposure: CVE-2021-43113 (critical, CVSS 9.8) and CVE-2017-9096 (high, 8.8). Remainder are medium: five jQuery UI, one Knockout.js, two CPython.

**Hosting (pp.28–29).** On-premises in third-party colocation — production Orlando, DR Atlanta, dedicated physical hardware. Three-tier DR: server failover 15–30 min RPO/RTO; both-live failure 24–48 hrs; worst case 7 days. Hosting OpEx ~$15k → $16k → $35k (2023–25), <1% of gross profit; the 2025 step-up ties to a ~$194k hardware refresh and expanded DR, not run-rate escalation. Management sees no near-term case for wholesale Azure migration and EY-P agrees. AI inference is outsourced (Groq), so no meaningful hosting/CapEx burden. **DR has never been formally tested end-to-end** — process-dependent rather than institutionalized.

## Data and AI (pp.31–33)

Assessed as the strongest area — "on target or exceeding target" across data sources, pipeline, and algorithms.

- Ingestion: SFTP ~70% of incoming client data; XML webhooks limited to ISO/Verisk; manual UI entry.
- Screening funnel: deterministic rules → **BioBERT** (ICD-10 / clinical feature extraction) → **XGBoost** (predictive ARC score). Both retrained regularly. Deterministic filtering before probabilistic scoring keeps inference cost and overhead down.
- LLM document intelligence: two-model split on GroqCloud — Llama 4 Scout for page classification, openai/gpt-oss-120b for reasoning and evidence extraction; outputs written back to claim records. Token-based pricing, minimal cost per document.
- Roadmap: intelligent work routing (predictive queue prioritization), expected pay modelling and variance detection, agentic task execution (first scoped to PHI-minimized Tort Recovery with human oversight).
- Data warehouse on the roadmap but not selected; reporting currently runs off the OLTP database.
- **Moat caveat:** most external data is commercially accessible (ISO/Verisk) or public (state crash reports, carrier websites). Differentiation comes from workflow-embedded manual tagging/enrichment and proprietary historical claim + ARC outcome data — not from exclusive data access. Coverage discovery remains manual and fragmented (employer DBs, workers' comp records, SoS registries, phone outreach, skip tracing), which caps scalability.

## R&D organization and SDLC (pp.35–36)

- 14 total R&D personnel vs. a benchmark of 43 for this size. All product development executed by **external contractors** — 10 across two vendors (Droit, long-tenured; Lapiz, added ~2.5 yrs ago for competition/redundancy). No internal dev or QA capability.
- The Director of IT also acts as Product Manager. No dedicated senior product or engineering leadership → reactive prioritization, delayed modernization decisions, reliance on contractor-led recommendations.
- R&D spend ~$900k (~9% of revenue) — judged appropriate and efficiently deployed. (p.15 says ~8%; minor internal inconsistency.)
- SDLC: Agile, two-week sprints, work split across Incidents and GPIs (multi-sprint epics). **No automated testing framework. No CI/CD — merges and releases are manual.** Source control on TFS per p.25 — but p.44 says Azure DevOps and p.36 lists both; unresolved contradiction. Visual Studio 2017. Multiple teams sometimes work parallel code paths requiring manual reconciliation.
- AI tooling in the SDLC: **Claude Teams** used broadly across design, requirements, research, and code review; **Google Antigravity** for agent-driven development in the Python AI/ML team. GitHub Copilot evaluated, not adopted.

## Cybersecurity (pp.41–46) — the most urgent workstream

**No dedicated cybersecurity resources.** Program led by Jatni Blandon (Director of IT) with oversight from Larrian Martin (CIO) and Rick Fossier (President). MDR/SOC outsourced to Kaseya (formerly RocketCyber).

In place: MFA for all users (Microsoft Authenticator), Datto AV/EDR + Microsoft Defender, BitLocker, Azure PIM, annual internal/external pen test (ERMProtect, April 2026 — one high-risk finding), Veeam v13 with ransomware immutability, documented multi-site DR, security awareness training (Mineral) and continuous KnowBe4 phishing simulations, documented IR plan with a 2026 ransomware tabletop.

Gaps, roughly in order of exposure:

- **ProWeb is internet-facing with no WAF**, storing ~1m PHI records.
- **No SIEM / single pane of glass** — telemetry fragmented across tools; Kaseya does not ingest cloud or ProWeb feeds.
- **Offshore third-party developers can push code directly to production.** No dynamic application security testing.
- **Six users hold standing admin roles**; JIT elevation is not the operating model. No application-layer RBAC.
- **No SSO** across the environment. Service accounts excluded from MFA (compensating controls: blocked interactive login, IP restrictions, password rotation).
- **USB ports not disabled, no DLP** — Purview DLP scoped to admin accounts only.
- Vulnerability scanning is ad hoc, not on a defined cadence. Windows Server 2016 approaching EOL. **EY requested but never received an asset inventory.**
- No immutability validation results provided; no backup schedule by system/retention.
- Cyber insurance: Coalition, $5m aggregate, no claims. WTW benchmark for peers is $10–15m ($200–375k premium) — **Granite is below benchmark**.
- IR retainer status unclear.
- EY-P notes leadership showed knowledge gaps on core cyber controls during interview.

**Incident discrepancy:** management stated five incidents; documentation provided later showed **seven** over five years. The pattern matters more than the count — two cross-client SFTP data exposures (June 2024, 571 non-client accounts; January 2026, 174 patients' SSN/DOB/contact data plus 11 misattached medical records) recurring from a first occurrence in 2022, indicating **no remedial controls were put in place after the first event**. Also two phishing credential-theft events (Jan 2025, Jul 2025) using the same vector, two misaddressed-attorney PHI disclosures (Aug and Sep 2025), and one insider-threat concern (Oct 2024). No major business impact to date; the Jan 2026 SFTP event is the most material.

## Total recommended investment

| Theme | One-time | Annual |
|---|---|---|
| Cybersecurity remediation | ~$205k–$290k | ~$70k–$100k (vCISO) |
| Technical debt | ~$365k–$485k | — |
| Process / SDLC | ~$30k–$50k | — |
| Organization (Digital Product Director + Director of Engineering) | — | ~$300k–$400k |
| Automation initiatives | ~$100k | ~$55k |
| **Total** | **~$700k–$925k** | **~$425k–$555k** |

Cyber subtotal is derived from the p.11 line items (compromise assessment ~$60–80k, WAF <$5k, data privacy assessment ~$125–175k, secure ProWeb ~$15–30k); EY-P does not state it. The ~$700k–$925k and ~$425k–$555k totals are EY-P's.

Automation upside: **~$620k annual savings (~$4.61/claim)** at current run-rate volumes. The roadmap covers only ~40% of identified opportunities; **60% is unplanned**. ~2 months to deliver, but full value realization is gated on ProWeb modernization.

## Sequencing and risks to carry forward

1. **Cybersecurity is the pre-announcement gate.** Test immutable backups, run a non-transaction phishing exercise, restrict removable media, and run a compromise assessment (~$60–80k) *before* announcement. WAF and transaction-related phishing exercise sign-to-close.
2. **Business logic extraction (XL) is the long pole** and everything ProWeb-adjacent in the automation roadmap sits behind it. 22–28 months is a long dependency chain for a hold period.
3. **The org gap is causal, not incidental.** EY-P links the absence of internal product/engineering ownership directly to the accumulated technical debt. Hiring profiles depend on the rebuild-vs-modernize-in-place decision, which is unresolved.
4. **The data moat is thinner than the AI story suggests** — value is in workflow-embedded enrichment and execution, both of which are labor, not software.
5. **Recurring cross-client data segregation failures** are the finding to press hardest: same failure mode in 2022, 2024, and 2026 means the control environment does not learn.

## Open questions

- SQL Server version is unconfirmed — the entire DB modernization path and 2027 deadline rest on an inference.
- No asset inventory was ever provided.
- DR has never been tested end-to-end; the stated RPO/RTO figures are design targets, not validated.
- Data warehouse and Windows OS upgrade are unsized, so the ~$700k–$925k one-time total is incomplete.
- Immutability of backups is asserted but unvalidated.
- IR retainer — in place or not.
- Is there a BAA with Groq covering PHI in the medical records sent for inference? Not addressed by EY-P.
- Expected hold period — unknown, and it should drive the rebuild-vs-modernize decision more than any technical factor.

## Related

- [[park-road-granite-platform-takeover-roadmap]] — proposed takeover roadmap, per-task risk register, and where we disagree with EY-P.
- Report is a **working draft** marked "PRELIMINARY DRAFT FOR REVIEW" throughout, dated May 12, 2026, intended solely for Park Road management and board.
