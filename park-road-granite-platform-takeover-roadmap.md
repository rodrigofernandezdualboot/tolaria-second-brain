---
type: Note
status: Draft
related_to:
  - "[[park-road]]"
  - "[[patterns-of-legacy-displacement]]"
---

# Park Road / Granite — Platform Takeover Roadmap & Risk Register

**Audience:** Dualboot internal. Candid about delivery risk to us.
**Premise:** post-close, Dualboot takes over ProWeb as the delivery partner.
**Source of record:** EY-P Project Granite TDD, 12 May 2026 (working draft). See [[park-road]].
**Method:** incremental displacement, per [[patterns-of-legacy-displacement]]. The Phase 2 gate is resolved in favour of displacement; the in-place refactor is retained as a documented fallback — reasoning in §3.
**Sizing:** T-shirt sizes on every task, calibrated in §4. No hour figures are attached to tasks by design.

---

## 1. The honest read

We would be taking over a platform where **the application is a passthrough and the product is a database**. All business logic — claim state, Next Actions, payer routing, coverage rules, eligibility — lives in T-SQL stored procedures and triggers. There is no automated test suite, no CI/CD, no reproducible build proof, no specification, and no internal engineering staff. The people who understand the system are contractors at a vendor (Droit) that also acts as final code controller.

That combination is the whole risk story. Every technical task below is harder than its label because we cannot yet answer the question *"if I change this, what breaks?"* — and neither can anyone else.

The good news is real: the Python AI/ML stack (BioBERT, XGBoost, Groq-hosted LLMs) is architecturally separate from ProWeb, well-maintained, and can be extended without touching the monolith. That is where early value lives and where we should want to be measured first.

---

## 2. Where we disagree with EY-P

Six things in their report would hurt us if we signed to them.

**2.1 The business logic refactor estimate is not credible as a commitment.**
EY-P sizes business logic refactoring at **1,600–2,200 hours** — roughly one person-year — derived from "contractor scrum team effort." That is to extract the operating logic of a platform running ~$9.5m of revenue, ~100 customers, and the full ARC claim lifecycle out of the database. We size it **XXL**, off the top of their scale. Our estimation reference is explicit that messy legacy with no tests and no docs should be treated at **90–100% of building from scratch**, not the 50% that "porting" intuitively suggests. Refuse to price it until Phase 0 produces the stored-procedure inventory.

**2.2 EY-P mis-sequenced the enabling investment.**
CI/CD and automated testing sit in their "Process & SDLC enhancements" bucket at S, described as improvements that "reduce delivery friction." They are not enhancements. **A characterization test harness is a hard prerequisite of any logic extraction**, it is absent from their plan entirely, and we size it **XL**. You cannot safely move logic out of stored procedures with no behavioral oracle to prove equivalence. This reordering is the single most important correction we make.

**2.3 The Windows Server 2016 deadline is much closer than "Low priority / phased."**
Verified against Microsoft lifecycle data: **Windows Server 2016 extended support ends 12 January 2027** and **SQL Server 2017 extended support ends 12 October 2027**. As of today (29 Jul 2026) that is **~5.5 months** and **~14.5 months**. EY-P's tech debt plan is **22–28 months**, they rate the OS upgrade Low priority, and they leave it unsized. Both platform floors go unsupported well before the modernization they are supposed to survive. With ~1m PHI records this is a HIPAA and cyber-insurance argument, not a patching inconvenience.

**2.4 The recurring data segregation failure is unsized and is probably a schema problem.**
Cross-client data exposure has happened **three times** — 2022, June 2024 (571 non-client accounts), January 2026 (174 patients' SSN/DOB/contact plus 11 misattached medical records). EY-P's own description of the 2026 event names the cause: *"account numbering collisions and SFTP processing logic."* Account identity is not reliably client-scoped. That is a data model defect touching record identity, not an SFTP script bug — which is why patching it twice did not hold. **If we take over operations, we inherit the fourth occurrence.**

**2.5 "Migrating to Azure SQL" is not a decision — it is two very different decisions.**
Azure SQL **Database** is PaaS with instance-level features removed; Azure SQL **Managed Instance** preserves them. Microsoft's [migration overview](https://learn.microsoft.com/en-us/data-migration/sql-server/database/overview) presents the loss of instance-scoped dependencies as a *benefit* — "eliminate any dependency on technical components that are scoped at the instance level, such as SQL Agent jobs." For a platform whose logic is entirely T-SQL, that is the bill, not a benefit. The same page confirms SQL Agent jobs unsupported (use elastic jobs), Windows logins unsupported (use Entra ID), only `master` and `tempdb` as system databases.

Nobody has inventoried Granite's SQL Agent jobs, linked servers, or cross-database queries, so nobody can say which target is viable. Microsoft's sequence is assessment first — **task 0.11**. Working position: `[ASSUMPTION]` Managed Instance is the realistic cloud target if we go to cloud at all, but in-place SQL Server 2022 is the right default — it buys the October 2027 runway at a fraction of the effort and defers the cloud decision until the architecture suits PaaS.

**2.6 Reporting is not a Phase 4 concern — it is what unfreezes everything else.**
EY-P leaves the data warehouse unsized and calls it not feasible near-to-medium term. That inverts the causality. Power BI queries the transactional database directly, so **every schema change during displacement is gated on operator-facing reports nobody has mapped.** This is an **Invasive Critical Aggregator**, and the symptoms match: processing coupled to source data structure, change control spreading from the aggregator outward to upstream systems, outputs bloating because extending a report beats creating one. Its treatment — **Divert the Flow** — builds a new decoupled implementation *early*, precisely so upstream is free to change. We move it from Phase 4 to the front of Phase 2. It is cheap here: Granite's aggregator is Power BI queries, not a mainframe batch job.

---

## 3. Method, and why the gate resolves toward displacement

Earlier drafts offered two co-equal Phase 2 options. Reading the displacement patterns against the diligence findings resolves it.

**The in-place path is literal Feature Parity on a system nobody understands.** Extracting the whole stored-procedure surface means reproducing every existing behavior — including bugs, workarounds, and dead paths — with no specification and nobody who can describe intended behavior. It also produces no user-visible value until late and does not remove the reporting coupling.

**The multi-product diagnosis fits Granite exactly.** Extract Product Lines argues that one physical system serving several logical products becomes over-generic, and observes that such systems characteristically have "very little in the way of automated test coverage, instead relying on huge, often manual regression suites." Granite runs four solution lines on one ProWeb with no automated testing and manual onshore UAT.

Phase 2 therefore uses this stack:

| Pattern | Where it applies at Granite |
|---|---|
| **Critical Aggregator** (anti-pattern) | Power BI on the ProWeb OLTP database. The thing to displace first. |
| **Divert the Flow** | New decoupled reporting/analytics implementation, then retire the direct-to-OLTP path. Unfreezes the schema. |
| **Extract Product Lines** | Decomposition axis: Discovery, Aged, Historical, Day 1. Slice vertically by solution line, not by technical layer. |
| **Revert to Source** | ~70% of ingestion is client SFTP; EDI and placement files originate outside ProWeb. New components read closer to source — including the AI/ML pipeline, which today reads screened claims from ProWeb. |
| **Event Interception** | ISO/Verisk XML webhooks and SFTP arrivals, where direct source integration is unavailable. |
| **Legacy Mimic** | Keep ProWeb's outward behavior stable for clients, clearinghouses, and ISO/Verisk while internals move. |
| **Transitional Architecture** | Façade, CDC, reconciliation scaffolding. Budget its **removal**, not just its build. |
| **Feature Parity** (as warning) | Chase the outcomes each solution line delivers, not ProWeb's feature list. |

### Two design constraints that are not optional

**The analytical model in 2A is a target model, not a copy.** Lift-and-shift CDC that replicates ProWeb's schema 1:1 into a new store is the path of least resistance and it defeats the purpose: reports stay coupled to ProWeb's shape at one remove, so when a 2B service diverges, the mirror breaks and the reports break with it. The model must be expressed in business concepts — claim, payer, coverage event, recovery — decided before the services that own them exist. This is what makes 2A harder than it sounds, and it is the difference between 2A absorbing 2B's schema churn and merely postponing it.

**Service granularity is a decision, not a default.** Extract Product Lines argues for slicing by product line and valuing use over reuse; it does not mandate microservices. Granite has 14 R&D people, all contractors, no internal dev or QA, and no CI/CD today. A distributed estate with per-service data stores adds operational burden they have no capacity to absorb, and the Director of Engineering hire has not happened. Four modular services with clear data ownership — or a modular monolith per line — may be right for a business this size. Decide deliberately.

**The in-place refactor is retained as a fallback**, documented at the end of §4, for the case where 0.13 shows the product lines cannot be separated.

---

## 4. Sizing scale

Sizes are calibrated against our internal `estimation-reference` and expressed on bands compatible with EY-P's own S/M/L/XL usage (p.25–26), so our numbers can be compared to theirs directly.

| Size | Band | EY-P anchor |
|---|---|---|
| **XS** | Days. One person, one sitting to one week. | — |
| **S** | Weeks. One or two people, under a month. | Source code management; CI/CD (180–300h) |
| **M** | A month or two of a small team. | .NET modernization (400–700h); infrastructure (700–800h) |
| **L** | A quarter of a small team. | Frontend / UI modernization (820–1,000h) |
| **XL** | Multiple quarters, or a dedicated team for a quarter-plus. | Business logic refactoring (1,600–2,200h) |
| **XXL** | Beyond EY-P's scale. Cannot be committed to as a single unit — must be decomposed and priced per part. | — |

**Calibration notes that matter more than the bands:**

- **Every "push to HIGH end" signal in the reference fires on this project**: legacy dependency, unclear requirements, high security/compliance (HIPAA, ~1m PHI records), and new team formation. There is no signal pushing us low. Sizes below sit at the top of their ranges by default, not the midpoint.
- **The porting rule applies at its harshest setting.** The reference grades ported features by legacy quality: well-structured with tests at 50–60% of new build, average without tests at 70–80%, and *messy legacy with no tests and no docs at 90–100%*. Granite is the third case. Treat displaced functionality as new build.
- **No AI velocity credit is applied.** The reference gates its adjustment rates on `stackFamiliarity = Confirmed`. For .NET Framework 4.5, T-SQL-resident business logic, Knockout, and Kendo UI, our familiarity is Unknown at best — which the modifier table maps to **0% across the board**. Even under full calibration, complex business logic caps at −5 to −10%, and PM and code review are always 0%. Anyone shaving these sizes on AI-tooling grounds is misapplying the reference.
- **Several sizes are count-driven and we do not have the counts.** Report count (0.9), WCF endpoint count, stored-procedure and trigger counts (0.2), and domain module count all move their tasks by a full size or more. These are marked ⚠ below and are the reason Phase 0 exists.
- Sizes are **provisional pending Phase 0** and deliberately carry no hours. The reference's own instruction applies: never copy ranges without reasoning from project-specific data, and we do not have that data yet.

---

## 5. Proposed roadmap

### Phase 0 — Establish control and buy information (ideally starts pre-close)

Converts unpriced risk into priced risk. Push to run as much as access allows *before* signing a delivery scope.

| # | Task | Size | Why it exists |
|---|---|---|---|
| 0.1 | Clean-room build validation — clone, build ProWeb from scratch, deploy to a throwaway environment | **S** ⚠ | With no CI, nobody has proof the repo is complete and buildable. Highest-information test available. **If the build fails, this becomes L+ reconstruction work** — size it again at that point. |
| 0.2 | Static dependency map — stored procedures, triggers, views, SQL Agent jobs, linked servers, cross-database references | **M** ⚠ | Reference: legacy codebase analysis 8–24h per module, pushed high for poorly structured and untested code. Module count unknown. Feeds 0.11, 0.13, and all of Phase 2. |
| 0.3 | Behavioral baseline capture — production query/workflow logging to build the regression oracle | **S** | Setup is modest; the constraint is calendar, not effort. Must span a full month-end. Start day 1. |
| 0.4 | Environment and asset inventory; confirm SQL Server version, edition, licensing | **S** | EY requested an inventory and **never received one**. SQL Server 2017 is an inference. Licensing determines Azure Hybrid Benefit eligibility. |
| 0.5 | End-to-end DR failover test (Orlando → Atlanta) | **S** | Reference: cutover rehearsal 16–32h. Stated RPO/RTO of 15–30 min are design targets, never validated. |
| 0.6 | PHI data-flow map and BAA review — including Groq and offshore developer access | **M** | Reference: compliance work 40–80h per standard, legal review scoped separately. Medical records go to a third-party inference provider; offshore contractors push to production. |
| 0.7 | Root cause analysis of the cross-client segregation defect | **M** | Determines whether this is a script fix or a tenancy/schema change. Changes the shape of the whole roadmap. |
| 0.8 | Vendor, license, change-of-control review — Kendo UI/Telerik, Veeam, Datto, Kaseya, Mineral, KnowBe4, Coalition | **XS** | Largely legal and commercial rather than R&D effort. Route to Park Road's workstream. |
| 0.9 | Reporting dependency map — which Power BI reports bind to which OLTP objects, who owns each, real-vs-bloat | **M** ⚠ | Now a Phase 2A input, not just a constraint register. Divert the Flow warns to decompose *who uses which output* before rebuilding. Size scales directly with report count. |
| 0.10 | Vendor transition and knowledge-capture plan with Droit and Lapiz | **S** | Planning only; execution is separate and ongoing. Their knowledge is the only specification that exists. |
| 0.11 | **Azure Migrate assessment** — discovery and assessment before committing to any database target. Output: target recommendation, right-sizing, monthly cost, feature-compatibility gap list | **M** | **No database target gets chosen without this.** Must enumerate what Azure SQL Database drops: SQL Agent jobs (→ elastic jobs), linked servers, cross-database queries, Windows logins (→ Entra ID), system-database usage, and third-party agents needing OS access (Datto RMM/EDR, Veeam, the p.28 file server) — Microsoft's stated trigger for choosing SQL Server on Azure VM. Consumes 0.2 and 0.4. |
| 0.12 | Data flow map to ultimate source — trace each key flow past ProWeb to where it originates; capture what ProWeb discards or delays | **M** ⚠ | Revert to Source prerequisite. Architecture diagrams stop at legacy; the value is behind it. Also finds upstream data currently thrown away because ProWeb had nowhere to put it. |
| 0.13 | **Product line separability assessment** — can Discovery, Aged, Historical, and Day 1 be separated, or do they share one inseparable Next Actions engine? | **M** | Determines whether Extract Product Lines is viable and therefore which Phase 2 we are pricing. p.17 suggests Discovery activates only ARC detection and coverage discovery, which would make it a genuinely thin slice — confirm. |

**Phase 0 aggregate: M.** No single task above S–M, which is why this phase is cheap relative to what it de-risks.

**Exit criteria:** we can build and deploy ProWeb ourselves; object-level inventory exists; we know what our changes can break; evidence-based database target with costs; we know whether the product lines separate; 2.1 is priceable.

### Phase 1 — Make change safe and clear the 2027 wall

| # | Task | Size | Notes |
|---|---|---|---|
| 1.1 | Consolidate source control to Azure DevOps; branch policies; **revoke direct-to-production push for third-party developers** | **S** | Agrees with EY-P. Resolve the TFS-vs-Azure-DevOps contradiction (§8) first. |
| 1.2 | Reproducible build → CI → automated deploy to lower environments | **M** | **EY-P says S.** Reference puts CI/CD at 16–24h with on-premise adding 30–50%, but that assumes a build that already works. Here buildability is unproven (0.1) and the estate is TFS-era. Build reproducibility precedes pipeline automation. |
| 1.3 | **Characterization / golden-master test harness over the stored procedure surface** | **XL** ⚠ | **Absent from EY-P's plan.** This is effectively writing the specification that does not exist. Size scales with procedure and trigger count from 0.2. Gates all of Phase 2. |
| 1.4 | Masked or synthetic lower environment | **L** | **Absent from EY-P's plan.** Reference: ETL 8–20h per entity plus 16–40h validation, across many entities, with referential integrity preserved. Prerequisite for offshore development without offshore PHI. |
| 1.5 | .NET Framework 4.5 → 4.8.1 | **M** | Agrees with EY-P. Low architectural value, real security value. |
| 1.6 | **Windows Server 2016 → 2022** | **M** | **EY-P leaves this unsized and rates it Low.** Hard deadline 12 Jan 2027. Procure ESU regardless. |
| 1.7 | Dependency remediation — iTextSharp CVE-2021-43113 (CVSS 9.8) and CVE-2017-9096 (8.8) first, then jQuery UI / Knockout / CPython | **S** | Reference: security hardening 16–40h. **Becomes M if iTextSharp needs replacing rather than upgrading** — its licensing changed across major versions. |
| 1.8 | WAF in front of ProWeb; DAST in the pipeline | **S** | Agrees with EY-P's <$5k on the WAF. Monitor → tune → enforce. |
| 1.9 | Observability baseline — app and DB telemetry into a single monitoring plane | **M** | No SIEM today. Also supplies the change-impact signal Phase 2 depends on. |
| 1.10 | **Database target decision and execution — gated on 0.11.** Default: in-place SQL Server 2022 to clear 12 Oct 2027 | **L** in place / **XL** to cloud | **EY-P says M.** In-place is larger than a version bump because compatibility-level and deprecated-T-SQL behavior must be validated across thousands of procedures behind 1.3. Cloud adds a full migration: Azure DMS online mode or transactional replication, **not BACPAC** (export time rises sharply with object count and it requires downtime). Use vCore for Azure Hybrid Benefit. Scale target resources for cutover — transaction log rate is governed in Azure SQL Database. |

**Phase 1 aggregate: XL**, driven almost entirely by 1.3 and 1.10.

**Exit criteria:** we can change ProWeb and know within minutes if we broke it. Both 2027 floors cleared or formally deferred with ESU.

### Phase 2A — Divert the Flow: displace the aggregator first

Runs as soon as 1.3 and 0.9 allow. The opening displacement move, because it is what unfreezes the schema for everything after.

| # | Task | Size |
|---|---|---|
| 2A.1 | Decoupled analytical store fed by CDC from ProWeb, plus **Revert to Source** feeds for anything ProWeb only passes through (per 0.12). **Target model, not a ProWeb mirror** — see §3 | **L** |
| 2A.2 | Rebuild reports **iteratively, one at a time, to production** | **L** ⚠ Count-driven; 0.9 sets this |
| 2A.3 | **Parallel running with reconciliation** against existing Power BI outputs, with divergence alerting | **M** |
| 2A.4 | Hunt down **"off system" workarounds** — manual spreadsheet steps between ProWeb output and what leadership actually sees | **S** |
| 2A.5 | Stage cutover by user cohort; retire the direct-to-OLTP query path per report | **M** |
| 2A.6 | Fix data quality issues **upstream**, not by reimplementing legacy workarounds | **M** ⚠ Unbounded until 2A.3 findings land |

**Phase 2A aggregate: L.**

**Deliberately not feature parity.** 0.9's named-owner list is the scope gate; unowned reports are retired, not migrated.

**Expect the numbers to disagree.** Legacy reports commonly contain undiscovered bugs. Build worked examples with known inputs and business-agreed outputs *before* cutover, so divergence can be attributed rather than argued about.

**Exit criteria:** no operator-facing report queries the transactional database directly. From here, schema change is ours to make.

### Phase 2B — Extract Product Lines: displace by solution line

Gated on 0.13 confirming separability.

**Sequence: Discovery as a funded pilot, then Aged.** The pattern's heuristic says take the *second riskiest* line, pointing at Aged or Historical. We deviate deliberately: with no harness, no specification, and no internal engineering staff, the first slice buys knowledge, not revenue. Discovery is the thinnest viable slice — 2% of Next Actions volume, advisory-only, client retains billing and AR, and per p.17 it activates only ARC detection and coverage discovery.

The heuristic's warning still binds: a 2% win will not sustain sponsorship across a multi-year programme. So **Aged is named upfront**, and Discovery is budgeted explicitly as pattern-establishing work — scaffolding, harness, deployment path, team shape. If we cannot say what Discovery produced that makes Aged cheaper, the pilot failed.

| # | Task | Size |
|---|---|---|
| 2B.1 | **Discovery (pilot).** New service implementation; **Event Interception** on ISO/Verisk webhooks and SFTP arrivals; **Revert to Source** for placement files. Deliverable includes reusable scaffolding, not just the feature | **L** |
| 2B.2 | **Legacy Mimic** layer so clients, clearinghouses, and ISO/Verisk cannot tell which system serves them | **L** ⚠ Endpoint-count-driven; reference puts compatibility shims at 4–10h per endpoint |
| 2B.3 | **Aged** — first real line, 32% of volume, 20–30% of collections. Staged cutover: beta cohort, then 1% / 5% / 10% | **XL** |
| 2B.4 | **Historical** — 25% | **XL** |
| 2B.5 | **Day 1** — 42%, full lifecycle ownership, last | **XXL** |
| 2B.6 | Progressive retirement of displaced ProWeb modules, contracted as named deliverables | **M** |
| 2B.7 | **Removal of transitional architecture** — façade, CDC bridges, reconciliation jobs | **L** |

**Phase 2B aggregate: XXL.** Not commitable as a unit. Price per product line, after 0.2 and 0.13.

Per Extract Product Lines: identify shared capabilities early and **value use over reuse**. Duplication between lines is the intended trade-off, not a defect.

### Phase 3 — Automation and AI enablement

Runs in parallel throughout, because it does not depend on ProWeb.

| # | Task | Size |
|---|---|---|
| 3.1 | Triage the **60% unplanned** automation opportunity by ProWeb dependency and automation *class* (see R8) | **S** |
| 3.2 | Ship off-platform automation now — the Python/Groq pipeline needs no ProWeb change | **L** |
| 3.3 | Intelligent work routing; expected pay modelling; variance detection | **L** |
| 3.4 | Agentic task execution — Tort Recovery, PHI-minimized, human oversight, bounded action library | **L** |
| 3.5 | Payer portal and telephony automation | **XL** + ongoing ⚠ Separate risk class — see R8 |
| 3.6 | Point the AI/ML pipeline at source data rather than ProWeb SQL where 0.12 shows ProWeb only passes data through | **M** |

### Phase 4 — Data platform maturity

Reduced, because reporting displacement moved to 2A.

| # | Task | Size |
|---|---|---|
| 4.1 | Extend the 2A analytical store into longitudinal claim analytics | **M** |
| 4.2 | Feature store for the ML pipeline | **M** |

### Fallback — in-place refactor

If 0.13 shows the product lines cannot be separated. Phase 2A still comes first and still applies.

| Task | Size |
|---|---|
| Domain-by-domain logic extraction from stored procedures behind the 1.3 harness | **XXL** (EY-P: XL) |
| Trigger elimination — convert implicit DML side effects to explicit service calls | **XL** (EY-P: no line item) |
| WCF → REST replacement (insurance and fax workflows) | **L** + partner-calendar risk |
| .NET 4.8.1 → current .NET, blocked on WCF | **L** |
| Frontend: incremental React islands replacing Knockout/Kendo | **L** (agrees with EY-P) |

---

## 6. Our sizing vs EY-P's

The comparison, not the individual numbers, is the finding.

| Item | EY-P | Ours | Delta and why |
|---|---|---|---|
| Business logic refactoring | XL | **XXL** | Messy legacy, no tests, no docs → 90–100% of new build, not 50%. §2.1 |
| .NET framework modernization | M | M | Agree |
| Frontend / UI modernization | L | L | Agree |
| Source code management | S | S | Agree |
| WAF | <$5k | S | Agree |
| CI/CD pipeline | S | **M** | On-premise premium, plus buildability is unproven |
| Infrastructure / database | M | **L**–**XL** | Compatibility validation across thousands of procedures; cloud adds a full migration |
| Windows OS upgrade | **unsized**, rated Low | **M** | Deadline is ~5.5 months away |
| Data warehouse | **unsized**, "not feasible near-to-medium term" | **L**, as Phase 2A | Reframed from analytics maturity to schema prerequisite. §2.6 |
| Characterization test harness | **absent** | **XL** | Prerequisite of everything else. §2.2 |
| Masked / synthetic environment | **absent** | **L** | Offshore PHI exposure blocks our delivery model without it |
| Legacy Mimic layer | **absent** | **L** | Partner-facing behavior must not change during displacement |
| Trigger elimination | **absent** | **XL** | Top technical hazard, unnamed in the report |
| Product line displacement | **absent** | **XL**–**XXL** per line | Their plan has no displacement concept at all |

**The pattern:** EY-P sized the work that was visible in the code and omitted the work that makes changing the code safe. Four of our five largest items — harness, masked environment, trigger elimination, and per-line displacement — have no line item in their plan. Their ~$305k–$405k tech debt figure is internally consistent with the scope they described; the scope is what is wrong with it.

---

## 7. Risk register by task

Scored L(ikelihood) × I(mpact), H/M/L.

### Phase 0

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 0.1 Build validation | Repo incomplete — missing config, undocumented deploy steps, binaries not in source control, a build that only works on one contractor's machine | **H×H** | Any "ask Droit for that file" during the attempt | Week 1. A failed clean-room build is a go/no-go input, not a defect to fix later. Budget for reconstruction. |
| 0.2 Dependency map | Tooling cannot resolve dynamic SQL. Logic assembled at runtime in strings is invisible to static analysis | **H×M** | High `EXEC(@sql)` / `sp_executesql` density | Combine static analysis with runtime capture from 0.3. Report coverage as a percentage — never imply the map is complete. |
| 0.3 Behavioral baseline | Capture window misses month-end, quarter-end, payer-cycle edge cases; harness built on a partial picture | **H×M** | Coverage gaps against the p.18 workflow map | Span at least one full month-end. Start day 1 — calendar time we cannot compress later. |
| 0.4 Asset inventory | Still not provided, or SQL Server is older / lower edition than assumed | M×M | Repeated deferral, as happened with EY | Escalate as a close condition. Pre-2017 changes 1.10 timeline and cloud options. |
| 0.5 DR test | Test fails, or is refused because nobody will authorize a live failover | **M×H** | Reluctance to schedule a window | A refusal is itself the finding — report it to Park Road. Do not accept untested RPO/RTO as fact. |
| 0.6 PHI / BAA | No BAA with Groq covering medical records; offshore production PHI access unpapered | **M×H** | Vague answers on where inference runs | Legal review before we touch anything. A BAA gap is a pre-close issue for Park Road, not our remediation item. |
| 0.7 Segregation RCA | Root cause is record identity, implying schema change across the core data model | **M×H** | Account keys not client-scoped; collisions possible by construction | Size both options (compensating controls vs. identity re-model). Feeds 0.13 and 2B. |
| 0.8 Licenses | A commercial component has a change-of-control or per-seat restriction that bites post-close | M×M | Missing license documentation in the VDR | Route to Park Road's legal workstream while it is still their cost. |
| 0.9 Reporting map | Report owners cannot be identified, so real outputs cannot be told from bloat and we rebuild everything | **H×M** | Reports with no named owner; Power BI datasets querying tables directly | Named owner per report is the deliverable. Unowned reports are retirement candidates. |
| 0.10 Vendor transition | Droit reads the takeover as displacement and disengages, taking the only system knowledge | **M×H** | Slow responses, gatekeeping on repo access | Design them in, not out — paid knowledge transfer with named deliverables. |
| 0.11 Azure Migrate | Output treated as a recommendation rather than an input. Azure Migrate optimizes for a supported landing zone; it has no view on whether moving *now* is wise | **M×M** | A target selected on tool output alone | Assessment answers which targets are viable and at what cost. Whether and when stays our judgment. Record separately. |
| 0.11 | Assessment misses dependencies — jobs on unmanaged servers, ad-hoc linked servers, undocumented ETL | **M×H** | Discovery scope narrower than the 0.4 inventory | Reconcile 0.11 output line-by-line against 0.2 and 0.4. Any gap is unpriced migration risk. |
| 0.11 | Cost estimate built on current sizing, which reflects a database doing application work. The profile changes after displacement | M×M | Sizing taken from peak CPU on the current box | Decision input with a stated shelf life. Re-run during 2B. |
| 0.12 Source mapping | Upstream tracing stalls because the people who built the integrations have left | **M×M** | Nobody can explain how a feed arrives | Switch to downstream tracing from the integration point forward — the recommended fallback when legacy knowledge is lost. |
| 0.12 | Source systems turn out not reliable or available enough to feed us directly, found after we design for it | M×H | No availability data on client SFTP endpoints or the ISO/Verisk feed | Check cross-functional constraints on each source before designing Revert to Source into a slice. |
| 0.13 Separability | The four lines share one inseparable Next Actions engine, so Extract Product Lines does not apply | **M×H** | Same tables and procedures serving all four lines with only flag-based variation | The most consequential Phase 0 finding. Run early enough that the answer precedes any Phase 2 commitment. |

### Phase 1

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 1.1 Source control | History lost, or two sources of truth diverge | M×M | The TFS/ADO contradiction in §8 unresolved | Resolve current state first. Freeze, migrate, verify, cut over — no dual-write period. |
| 1.2 CI | Build automated but deploy still manual and business-coordinated, so release risk is unchanged | M×M | "We'll wire deploy up later" | Definition of done is deploy-to-lower-environment, not green build. |
| 1.3 **Test harness** | Coverage thinner than believed. Team gains confidence the harness does not justify | **H×H** | Coverage measured in tests written rather than behaviors pinned | Coverage per business workflow, not per procedure. No slice enters Phase 2 until its behaviors are pinned. **This risk decides whether the programme succeeds.** |
| 1.4 Masked environment | Masking breaks referential integrity, so it cannot reproduce real defects and gets abandoned | **M×M** | Developers asking for a production restore | Deterministic, referentially consistent masking. Validate by reproducing three historical production defects. |
| 1.5 .NET 4.8.1 | Serialization, TLS, or date-handling differences surface weeks later | M×M | Sparse harness coverage on money and date paths | Sequence after 1.3. Prioritize harness coverage on financial calculations. |
| 1.6 **Windows Server 2022** | We miss 12 Jan 2027; ProWeb runs unsupported holding ~1m PHI records | **H×H** | ESU not procured by Nov 2026 | Procure ESU as insurance now regardless of plan confidence. Test legacy .NET/IIS compatibility on 2022 early. |
| 1.7 iTextSharp | Upgrade is not drop-in — iText licensing changed across major versions (AGPL/commercial) | **M×M** | Version pinned very old for a reason | Check licensing alongside the CVE fix. Consider replacing rather than upgrading. |
| 1.8 WAF | Blocking mode breaks legitimate claim submission; gets disabled under operational pressure | **M×M** | No monitor-mode learning period | Monitor → tune → enforce. Never straight to blocking on a revenue-path application. |
| 1.9 Observability | Telemetry added, nobody owns alerts, noise leads to it being ignored | M×M | No named owner | Tie to the vCISO / Director of Engineering hire. Tooling without ownership is theatre. |
| 1.10 Database target | **Azure SQL Database chosen and SQL Agent jobs, linked servers, or cross-database queries break late in migration** | **M×H** | A target named before 0.11 completes | Hard gate on 0.11. Managed Instance preserves instance-level features. Default to in-place SQL Server 2022. |
| 1.10 | Cloud migration and displacement run concurrently, so two foundations move at once and neither rolls back cleanly | **M×H** | Cloud migration scheduled inside the Phase 2 window | Sequence, do not parallelize. Clear 2027 in place; move when the architecture suits it. |
| 1.10 | In-place upgrade surfaces deprecated T-SQL or compatibility-level behavior changes across thousands of procedures | **M×H** | No harness coverage on affected procedures | Data Migration Assistant first. Pin compatibility level, raise deliberately behind 1.3. |

### Phase 2A — Divert the Flow

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 2A.1 Analytical store | **Model becomes a 1:1 ProWeb mirror**, so reports stay coupled to ProWeb's shape at one remove. When a 2B service diverges, the mirror breaks and reports break | **H×H** | CDC configured table-for-table; no business-concept model defined | Define the target model in business terms before building feeds. This is the difference between absorbing 2B's schema churn and postponing it. See §3. |
| 2A.1 | Fed exclusively by CDC from ProWeb, inheriting ProWeb's data losses and batch timings | **M×H** | No Revert to Source feeds in the design | Use 0.12. Where ProWeb only passes data through, take it from source and gain the fields and timeliness ProWeb discards. |
| 2A.2 Report rebuild | Feature parity creeps in and we rebuild every bloated report, including unused ones | **H×M** | Reports in scope equals reports in existence | 0.9's named-owner list is the scope gate. Unowned means retired. |
| 2A.3 Reconciliation | New outputs do not match legacy, and the mismatch reads as our defect when it is a legacy bug | **H×M** | No agreed expected-output examples before cutover | Build worked examples with business-agreed outputs *before* parallel running, so divergence can be attributed. |
| 2A.4 Off-system workarounds | Leadership's actual numbers come from undocumented manual steps, so new reports are "wrong" in ways we cannot reproduce | **M×H** | Anyone spending days per month producing a report the system supposedly produces | Ask directly and early, without blame. Nobody volunteers this because nobody wants to tell leadership the reporting does not work. |
| 2A.5 Cutover | Reports retired before every consumer is found; a downstream spreadsheet or client deliverable breaks | **M×M** | Consumers identified by system rather than by person | Cohort-based cutover with a rollback window per report. |
| 2A.6 Upstream fixes | Data quality patched in the new store because fixing upstream is slower, recreating legacy workarounds in new code | **M×M** | Transformation logic accumulating special cases | Every new special case becomes a defect ticket against the upstream system. |
| 2A | **Phase deprioritized as "just reporting"** against louder platform work, so the schema stays frozen and 2B stalls | **M×H** | 2A slipping while 2B slices get scheduled | Frame 2A internally and externally as the schema unlock, not an analytics project. Nothing in 2B is safe until it lands. |

### Phase 2B — Extract Product Lines

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 2B.1 Discovery pilot | Delivers a feature but no reusable scaffolding, so Aged re-learns everything and the pilot's justification evaporates | **M×H** | Pilot scoped as a feature rather than scaffolding | Contract the scaffolding as the named deliverable. Acceptance test: state what makes Aged cheaper. |
| 2B.1 | A 2% slice fails to hold executive attention; funding softens before Aged starts | **M×H** | Sponsor engagement dropping after the pilot demo | Name Aged upfront with a date. This is the known cost of deviating from the second-riskiest heuristic — manage it, do not ignore it. |
| 2B.2 **Legacy Mimic** | Mimic incomplete and a partner integration notices — ISO/Verisk, a clearinghouse, or a client's SFTP expectations | **M×H** | Partner-facing contracts not inventoried before first cutover | Inventory external consumers in 0.12. Partner-visible changes are calendar-driven and outside our control. |
| 2B.2 | Mimic becomes permanent because keeping it is easier than retiring it, quietly becoming the new legacy | **M×M** | No retirement date attached at build time | Transitional Architecture discipline: every mimic component gets a removal owner and date at creation. |
| 2B.3 Aged cutover | Staged traffic shift lacks a real rollback path, so 10% becomes all-or-nothing under pressure | **M×H** | No tested rollback at the 1% stage | Prove rollback at 1% before going to 5%. The gradient only works if each step is reversible. |
| 2B.3 | Aged carries the recovery economics (20–30% of collections). A defect here has direct client revenue consequence | **M×H** | Harness coverage on Aged workflows below Discovery's | Do not start Aged until its behaviors are pinned to at least the standard Discovery reached. |
| 2B.4–5 | Shared capabilities accumulate between lines and we rebuild the over-generic system we were escaping | **H×M** | A shared services layer growing with each new line | Value use over reuse. Duplication is the intended trade-off. |
| 2B.4–5 | **Service granularity defaults to microservices** and Granite's 14-person all-contractor R&D function cannot operate a distributed estate | **M×H** | Per-service data stores proposed before the Director of Engineering is hired | Decide granularity deliberately (§3). Modular services or a modular monolith per line may be right for this size of business. |
| 2B.6–7 | **Displacement stalls part-way. Two platforms run indefinitely** — worse than either endpoint | **M×H** | Retirement milestones slipping while new-build continues | Contract module retirement as named deliverables. No new line starts until the prior module is dark. Fund removal explicitly. |
| 2B | Segregation defect (0.7) inherited into new services because record identity was copied rather than re-modelled | **M×H** | New services reusing ProWeb account keys unchanged | Displacement is when identity can change; refactoring in place is when it cannot. This is the window. |

### Phase 3 — Automation and AI

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 3.1 Triage | The unplanned 60% is assumed to have the same unit economics as the planned 40% | **H×M** | A single blended savings figure | Re-baseline the ~$620k / ~$4.61-per-claim opportunity ourselves before committing. |
| 3.2 Off-platform | Groq dependency: single provider, hosted models that can be deprecated or repriced | **M×M** | No provider abstraction in the Python codebase | Abstract the inference interface. Cheap now, protects margin later. |
| 3.3 Routing / expected pay | Models trained on historical outcomes encode current process bias; measured lift fails to appear | M×M | No holdout or champion/challenger design | Insist on holdout groups. Do not let "AI-driven" substitute for measured lift. |
| 3.4 Agentic execution | An autonomous action creates a billing or compliance error at scale before anyone notices | **M×H** | Action library expanding faster than the audit trail | Bounded action library, action-level audit, reversibility, staged rollout by volume. Keep PHI-minimized scoping. |
| 3.5 **Payer portals / telephony** | The bulk of remaining manual cost is portal RPA plus voice — brittle, breaks on every payer UI change, per-payer maintenance forever | **H×H** | Estimated as "workflow automation" alongside deterministic work | Separate estimate, separate risk class, separate commercial treatment. Maintenance is the real cost. Consider clearinghouse/API routes before per-payer RPA. |
| 3.6 Revert to source | Repointing the ML pipeline shifts its input distribution and model performance moves without anyone attributing it to plumbing | **M×H** | No parallel scoring during the switch | Score both feeds in parallel and compare before cutover. Drift caused by an infrastructure change is very hard to diagnose after the fact. |

### Fallback path

| Task | What goes wrong | L×I | Mitigation |
|---|---|---|---|
| Logic extraction | Effort lands at a multiple of EY-P's figure and we are anchored to a number we did not produce | **H×H** | Price per domain after 0.2. Contract per domain. Never quote EY-P's figure as ours. |
| **Triggers** | Invisible DML side effects. New service-layer writes bypass or double-fire them, corrupting claim state | **H×H** | **Top technical hazard, above stored procedures.** Map fan-out exhaustively; convert to explicit calls before extracting dependent procedures. Silent corruption in a billing system is the worst available failure mode. |
| WCF | No .NET Core equivalent; CoreWCF covers part of the surface. Insurance and fax integrations are third-party contracts we cannot change unilaterally | **H×H** | Inventory every endpoint and consumer in Phase 0. Partner coordination is calendar-driven and outside our control. |
| .NET Core | Hard-blocked on WCF, plus System.Web-coupled Razor views and the ADO connector | **H×M** | Communicate the dependency chain. EY-P presents .NET Core as "also being considered" without noting it is gated. |
| Frontend | React islands beside Knockout/Kendo produce two state models and a worse UX than either | **M×M** | Migrate by whole screen, never by widget within a screen. |
| Parallel work | Extraction runs alongside contractor feature work and the two collide continuously | **H×M** | Domain ownership boundaries and a change freeze per domain under extraction. |

---

## 8. Top risks, ranked

Ranked by expected cost to Dualboot, not technical interest.

| # | Risk | Why it ranks here |
|---|---|---|
| **R1** | We anchor commercially to EY-P's business logic estimate | Direct margin exposure on the largest item. Self-inflicted and entirely avoidable. |
| **R2** | Change begins before the characterization harness is real | Silent data corruption in a claims platform serving ~100 customers. Existential to the engagement. |
| **R3** | Clean-room build fails / repo is incomplete | Week 1 if we look, month 6 if we do not. Cheap test, enormous information value. |
| **R4** | Product lines do not separate (0.13), forcing the fallback path | Collapses the Phase 2 shape and puts triggers back at the centre of the programme. |
| **R5** | 2A's analytical store is built as a ProWeb mirror | Silently voids the entire schema unlock. Looks like success until the first service diverges. |
| **R6** | Droit disengages and system knowledge is lost | The only specification of the system is in contractors' heads. |
| **R7** | Windows Server 2016 (12 Jan 2027) and SQL Server 2017 (12 Oct 2027) expire mid-programme | Compliance and insurance exposure with ~1m PHI records. EY-P rated this Low. |
| **R8** | Cross-client segregation defect recurs on our watch | Fourth occurrence, attributable to us as operator. Reputationally worse than any technical slip. |
| **R9** | The remaining 60% automation opportunity is portal/telephony RPA | Where the promised $620k lives, and the hardest class of work to sustain. |
| **R10** | Displacement stalls part-way and transitional architecture becomes permanent | Two platforms forever. The signature failure of this approach, and it arrives quietly. |
| **R11** | Phase 2A deprioritized as "just reporting", so the schema never unfreezes | Blocks 2B without ever appearing as a blocker on a plan. |
| **R12** | A database target committed before the Azure Migrate assessment, or cloud migration concurrent with displacement | Azure SQL Database removes exactly the instance-level features a T-SQL platform runs on. Two moving foundations removes rollback. |
| **R13** | Service granularity defaults to microservices beyond Granite's operating capacity | 14 contractors and no internal QA cannot run a distributed estate. |
| **R14** | Granite hires a Director of Engineering who re-litigates our scope | Structural commercial risk. Anticipate it; ideally influence the hire. |
| **R15** | Offshore PHI exposure without a masked environment | Blocks our own delivery model until 1.4 lands. |

---

## 9. Contradictions in the source report to resolve

The report is a working draft marked "PRELIMINARY DRAFT FOR REVIEW" throughout and does not fully reconcile with itself. Each changes our scope:

- **Source control:** p.25 says **TFS** with Azure DevOps "evaluated but not initiated." p.44 says Granite "leverages **Azure DevOps** for source control." p.36 lists TFS, Azure Repos/Git, *and* Azure Repos/TFVC. `[ASSUMPTION]` the Python AI/ML repo is on Azure DevOps while ProWeb remains on TFS — confirm, since it determines whether 1.1 is a migration or a consolidation.
- **Inference runtime:** p.31 refers to "**Ollama**-based open source models" while the same page's diagram and p.32 specify **GroqCloud** with Llama 4 Scout and gpt-oss-120b. Ollama is local; Groq is hosted. Either there is local inference infrastructure nobody described, or this is a drafting error.
- **"Azure SQL"** used without specifying Database vs. Managed Instance (p.25). See §2.5.
- **R&D as % of revenue:** ~9% on pp.7 and 9, ~8% on p.15.
- **Incident count:** management said five, documents showed seven. Sets a baseline for weighting unverified management statements.
- **SQL Server version:** inferred, never confirmed.
- p.26 footnote markers do not match the footnote list. Minor, but consistent with a document whose numbers may still move.

---

## 10. What we need before we price anything

1. Stored procedure and trigger inventory with complexity metrics (0.2) — sets 1.3 and the fallback size
2. Successful clean-room build (0.1)
3. **Product line separability answer (0.13)** — determines which Phase 2 we are pricing
4. Report count and named owners (0.9) — sets 2A.2
5. WCF endpoint count and external consumers — sets 2B.2 and the fallback
6. Confirmed SQL Server version, edition, licensing (0.4)
7. Azure Migrate assessment output (0.11)
8. Data flow map to ultimate source (0.12)
9. Segregation defect root cause (0.7)
10. Droit transition terms (0.10)
11. Close date — everything in the 2027 analysis is relative to it

**Commercial posture:** T&M or capped-T&M for Phase 0, Phase 1, and the Discovery pilot. Fixed price only per product line, and only after 0.2 and 0.13. **Nothing sized XXL is commitable as a unit** — that is what the size means. We should be willing to walk away from a fixed-price commitment on full logic extraction, and say so early rather than discovering our position under pressure later.

---

## 11. Open questions

- Is our role remediation delivery, modernization execution, post-close R&D capacity, or a second-opinion read for Park Road? `[ASSUMPTION]` this note assumes delivery takeover.
- Do we displace Droit/Lapiz, sit alongside them, or absorb them?
- What is the expected hold period? A Discovery → Aged → Historical → Day 1 sequence needs a horizon we do not know.
- Does Park Road expect the ~$620k automation savings in the model? If underwritten, R9 becomes their risk and we should say so before close.
- Who owns the fourth segregation incident if it happens after we take over? Answer contractually.
- Is there a BAA with Groq covering PHI in medical records?
- Does Park Road have an existing Azure footprint or enterprise agreement? Changes the 0.11 cost case.
- Would Park Road support us influencing the Director of Engineering hire?
- Do we have internal comparables for a T-SQL-to-service-layer extraction at this scale? Without one, XXL is a judgment rather than a calibrated figure.

---

## Verification note

Microsoft lifecycle dates in §2.3 and R7 verified against Microsoft Learn on 29 Jul 2026: [SQL Server 2017](https://learn.microsoft.com/en-us/lifecycle/products/sql-server-2017) extended end 2027-10-12; [Windows Server 2016](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2016) extended end 2027-01-12. Azure migration guidance from [Migration overview: SQL Server to Azure SQL Database](https://learn.microsoft.com/en-us/data-migration/sql-server/database/overview) — that page covers **Azure SQL Database only** and does not address Managed Instance, so the Database-vs-MI comparison is reasoned rather than sourced. Displacement patterns verified against source; four pattern pages read in full (Critical Aggregator, Divert the Flow, Extract Product Lines, Revert to Source), the other four summarized from cross-references — see [[patterns-of-legacy-displacement]] for status and correction log. Sizing in §4–5 derived from the internal `estimation-reference` (task-type ranges, migration-specific ranges, porting-vs-new-build rule, AI velocity gating); per that reference's own instruction, ranges were not applied mechanically and **all sizes are provisional pending Phase 0**. Items marked ⚠ are count-driven with counts unknown. All Granite-specific figures trace to the EY-P report at page numbers cited in [[park-road]]. Every inference is tagged `[ASSUMPTION]`.
