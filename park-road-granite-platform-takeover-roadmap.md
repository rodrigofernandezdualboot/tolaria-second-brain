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
**Criticality:** every task is marked Critical or Non-critical against the test defined in §4.

---

## 1. The honest read

We would be taking over a platform where **the application is a passthrough and the product is a database**. All business logic — claim state, Next Actions, payer routing, coverage rules, eligibility — lives in T-SQL stored procedures and triggers. There is no automated test suite, no CI/CD, no reproducible build proof, no specification, and no internal engineering staff. The people who understand the system are contractors at a vendor (Droit) that also acts as final code controller.

That combination is the whole risk story. Every technical task below is harder than its label because we cannot yet answer the question *"if I change this, what breaks?"* — and neither can anyone else.

The good news is real: the Python AI/ML stack (BioBERT, XGBoost, Groq-hosted LLMs) is architecturally separate from ProWeb, well-maintained, and can be extended without touching the monolith. That is where early value lives and where we should want to be measured first.

---

## 2. Where we disagree with EY-P

This is the substance of our point of view. EY-P's report is competent, but six things in it would hurt us if we signed to them.

**2.1 The business logic refactor estimate is not credible as a commitment.**
EY-P sizes business logic refactoring at **1,600–2,200 hours / ~$140k–$190k** — roughly one person-year — derived from "contractor scrum team effort." That is to extract the operating logic of a platform running ~$9.5m of revenue, ~100 customers, and the full ARC claim lifecycle out of the database. We should treat that figure as a **floor, not an estimate**, and refuse to price it until Phase 0 produces a stored-procedure inventory (object count, lines of T-SQL, cyclomatic complexity, trigger fan-out). `[ASSUMPTION]` Our expectation is a materially larger number; we should range-check against internal comparables rather than inventing one here.

**2.2 EY-P mis-sequenced the enabling investment.**
CI/CD and automated testing sit in EY-P's "Process & SDLC enhancements" bucket at **$15k–$25k each**, described as improvements that "reduce delivery friction." They are not enhancements. **A characterization test harness is a hard prerequisite of any logic extraction.** You cannot safely move logic out of stored procedures with no behavioral oracle to prove equivalence. Any plan that runs extraction before the harness is a plan to break claims processing for 100 customers. This reordering is the single most important correction we make to their roadmap.

**2.3 The Windows Server 2016 deadline is much closer than "Low priority / phased."**
EY-P rates the OS upgrade **Low** priority with a "~3–12 months (phased)" timeline and leaves it unsized. Verified against Microsoft lifecycle data: **Windows Server 2016 extended support ends 12 January 2027** and **SQL Server 2017 extended support ends 12 October 2027**. As of today (29 Jul 2026) that is **~5.5 months** and **~14.5 months**. EY-P's tech debt plan is **22–28 months**. Both platform floors go unsupported well before the modernization they are supposed to survive. For a business holding ~1m PHI records, running unsupported OS and DB is a HIPAA and cyber-insurance argument, not just a patching inconvenience. This is a compound 2027 wall and it should be Phase 1, not "phased, low."

**2.4 The recurring data segregation failure is unsized and is probably a schema problem.**
Cross-client data exposure has now happened **three times** — 2022, June 2024 (571 non-client accounts), January 2026 (174 patients' SSN/DOB/contact plus 11 misattached medical records). EY-P's own description of the 2026 event names the cause: *"account numbering collisions and SFTP processing logic."* Account identity is not reliably client-scoped. That is a data model defect touching record identity, not an SFTP script bug — which is why patching it twice did not hold. Nobody has sized fixing it, and the regulatory and contractual exposure follows whoever operates the platform. **If we take over operations, we inherit the fourth occurrence.**

**2.5 "Migrating to Azure SQL" is not a decision — it is two very different decisions.**
EY-P frames the database path as "either upgrading to a supported SQL Server version or migrating to Azure SQL" (p.25). *Azure SQL* is not one product. Azure SQL **Database** is PaaS with instance-level features removed; Azure SQL **Managed Instance** preserves them. Microsoft's own [migration overview](https://learn.microsoft.com/en-us/data-migration/sql-server/database/overview) presents the loss of instance-scoped dependencies as a *benefit* — "you can then eliminate any dependency on technical components that are scoped at the instance level, such as SQL Agent jobs." For a platform whose business logic is entirely T-SQL, that is not a benefit, it is the bill. The same page confirms SQL Agent jobs are unsupported (use elastic jobs), Windows logins are unsupported (use Entra ID), and only `master` and `tempdb` exist as system databases.

Nobody has inventoried Granite's SQL Agent jobs, linked servers, or cross-database queries, so nobody can currently say which target is viable. Microsoft's recommended sequence is assessment first — Azure Migrate to discover and assess, then Azure Database Migration Service to move. **We adopt that sequence: task 0.11 below.** Our working position pending assessment: `[ASSUMPTION]` Managed Instance is the realistic cloud target if we go to cloud at all, but an in-place upgrade to SQL Server 2022 is the right default — it buys the October 2027 compliance runway at a fraction of the effort and defers the cloud decision until the architecture actually suits PaaS. EY-P separately concluded a broad Azure migration would raise infrastructure cost without clear operational benefit at current volumes, which supports the same conclusion from the cost side.

**2.6 Reporting is not a Phase 4 concern — it is the thing that unfreezes everything else.**
EY-P leaves the data warehouse unsized and calls it not feasible near-to-medium term, treating analytical separation as a maturity nice-to-have. That gets the causality backwards. Power BI queries the transactional database directly, which means **every schema change we might want to make during extraction is gated on operator-facing reports nobody has mapped.** In the vocabulary of [[patterns-of-legacy-displacement]] this is an **Invasive Critical Aggregator**, and the described symptoms match: processing coupled to source data structure, change control spreading from the aggregator to the upstream systems, and outputs bloating because extending a report is easier than creating one. Its recommended treatment — **Divert the Flow** — says build a new decoupled implementation *early*, precisely so the upstream is free to change. We therefore move reporting displacement from Phase 4 to the front of Phase 2. It is also cheap to divert here: Granite's aggregator is Power BI queries, not a mainframe batch job.

---

## 3. Method, and why the gate resolves toward displacement

Earlier drafts of this note offered two co-equal Phase 2 options: in-place logic extraction (Path A) or strangler-fig displacement (Path B). Reading the displacement patterns against the diligence findings resolves that.

**Path A is literal Feature Parity on a system nobody understands.** Extracting the whole stored-procedure surface into a service layer means reproducing every existing behavior — including the bugs, workarounds, and dead paths — with no specification and no one who can describe the intended behavior. That is the failure mode the pattern literature warns about most consistently. It also produces no user-visible value until late, and it does not remove the reporting coupling.

**The multi-product diagnosis fits Granite exactly.** Extract Product Lines argues that one physical system serving several logical products becomes over-generic, and observes that such systems characteristically have "very little in the way of automated test coverage, instead relying on huge, often manual regression suites." Granite runs four solution lines on one ProWeb with no automated testing and manual onshore UAT. The diagnosis lands.

So Phase 2 is displacement, using this stack:

| Pattern | Where it applies at Granite |
|---|---|
| **Critical Aggregator** (anti-pattern) | Power BI on the ProWeb OLTP database. The thing to displace first. |
| **Divert the Flow** | Build a new decoupled reporting/analytics implementation, then retire the direct-to-OLTP path. Unfreezes the schema. |
| **Extract Product Lines** | The decomposition axis: Discovery, Aged, Historical, Day 1. Slice vertically by solution line, not by technical layer. |
| **Revert to Source** | ~70% of ingestion is client SFTP; EDI and placement files originate outside ProWeb. New components should read closer to source rather than through ProWeb SQL — including the AI/ML pipeline, which today reads screened claims from ProWeb. |
| **Event Interception** | ISO/Verisk XML webhooks and SFTP arrivals, where direct source integration is not available. |
| **Legacy Mimic** | Keep ProWeb's outward behavior stable for clients, clearinghouses, and ISO/Verisk while internals move. |
| **Transitional Architecture** | The façade, CDC, and reconciliation scaffolding. Budget its **removal**, not just its build. |
| **Feature Parity** (as warning) | Chase the outcomes each solution line delivers, not ProWeb's feature list. |

**Path A is retained as a fallback**, documented at the end of §5, for the case where Phase 0 shows the product lines cannot be separated cleanly — for example if all four solutions prove to share a single inseparable Next Actions engine.

---

## 4. What "critical" means here

A task is **Critical** if at least one of these holds. The code in each table says which.

| Code | Test |
|---|---|
| **B** — Blocking | Other roadmap tasks cannot start, or cannot be sized, until this is done. |
| **G** — Gating | We cannot make a commercial commitment without it. |
| **D** — Deadline | A fixed external date applies and missing it has consequences we do not control. |
| **X** — Exposure | Skipping it leaves unacceptable legal, security, or data-integrity exposure while we hold responsibility. |

**Non-critical** means the task can slip or be deferred without breaking a dependency, missing a date, or creating exposure. **It does not mean optional or low-value.** Several non-critical items — the automation portfolio in Phase 3 above all — are where the client's return actually comes from.

**Where this binary strains.** Phase 3 is almost entirely non-critical by this test, because it deliberately has no ProWeb dependency. That is its virtue, and it is also the phase carrying the ~$620k business case. Read "non-critical" there as *non-blocking*, not *unimportant*. If the engagement is scoped for value delivery rather than modernization, the criticality ordering inverts and Phase 3 becomes the point.

---

## 5. Proposed roadmap

### Phase 0 — Establish control and buy information (target 4–6 weeks, ideally starts pre-close)

Cheap, fast, and it converts most of our unpriced risk into priced risk. We should push hard to run as much of this as access allows *before* signing a delivery scope.

**11 of 13 tasks are Critical.** That is by design — Phase 0 exists to answer gating questions, so almost nothing in it is deferrable.

| # | Task | Criticality | Why it exists |
|---|---|---|---|
| 0.1 | Clean-room build validation — clone the repo, build ProWeb from scratch, deploy to a throwaway environment | **Critical** (B, G) | With no CI, nobody has proof the repository is complete and buildable. Highest-information test available. Blocks 1.2 and 1.3; a failure is a go/no-go input. |
| 0.2 | Static dependency map — every stored procedure, trigger, view, SQL Agent job, linked server, and cross-database reference | **Critical** (B, G) | Sets the scope of 1.3, feeds 0.11 and 0.13, and is the only honest basis for pricing 2.1. |
| 0.3 | Behavioral baseline capture — production query/workflow logging to build the regression oracle | **Critical** (B, D) | Blocks 1.3. Also calendar-bound: the capture must span a full month-end, so a late start delays everything downstream and cannot be compressed. |
| 0.4 | Environment and asset inventory; confirm SQL Server version, edition, and licensing | **Critical** (B, G) | Gates 0.11 and 1.10. EY requested an inventory and **never received one**. SQL Server 2017 is an inference. Licensing determines Azure Hybrid Benefit eligibility. |
| 0.5 | End-to-end DR failover test (Orlando → Atlanta) | Non-critical | Blocks nothing and carries no date. Do it early anyway: it is cheap, and stated RPO/RTO of 15–30 min have never been validated. A refusal to authorize a live failover is itself a reportable finding. |
| 0.6 | PHI data-flow map and BAA review — including Groq and offshore developer access | **Critical** (X, G) | We would be handling ~1m PHI records, with medical records going to a third-party inference provider and offshore contractors pushing to production. A BAA gap is a pre-close issue for Park Road. |
| 0.7 | Root cause analysis of the cross-client segregation defect | **Critical** (B, X) | Feeds 0.13 and the 2B data model. Also a live recurring exposure that transfers to whoever operates the platform. |
| 0.8 | Vendor, license, and change-of-control review — Kendo UI/Telerik, Veeam, Datto, Kaseya, Mineral, KnowBe4, Coalition | Non-critical | Real risk, but it belongs to Park Road's legal workstream and blocks nothing of ours. Route it there while it is still their cost. |
| 0.9 | **Reporting dependency map** — which Power BI reports bind to which OLTP objects, who owns each, and which outputs are business-critical vs. bloat | **Critical** (B) | Sets the scope of 2A.2. Divert the Flow warns to decompose *who uses which output* before rebuilding, or you rebuild the bloat. |
| 0.10 | Vendor transition and knowledge-capture plan with Droit and Lapiz | **Critical** (B) | Droit's knowledge is the only specification that exists. Get this wrong and every subsequent task gets harder. See R5. |
| 0.11 | **Azure Migrate assessment** — discovery and assessment across the SQL estate before committing to any database target | **Critical** (G) | Gates 1.10. Must enumerate the instance-level dependencies Azure SQL Database drops: **SQL Agent jobs** (→ elastic jobs), **linked servers**, **cross-database queries**, **Windows logins** (→ Entra ID), system-database usage, and any third-party agent needing OS/file-system access (Datto RMM/EDR, Veeam, the p.28 file server) — Microsoft's stated trigger for choosing SQL Server on Azure VM instead. Consumes 0.2 and 0.4. |
| 0.12 | **Data flow map to ultimate source** — trace each key data flow past ProWeb to where it originates (client systems, EDI, ISO/Verisk), and capture what ProWeb discards or delays | **Critical** (B) | Prerequisite for Revert to Source in 2A.1, 2B.1, and 3.6, and for the external-consumer inventory that 2B.2 depends on. Architecture diagrams stop at legacy; the value is behind it. |
| 0.13 | **Product line separability assessment** — can Discovery, Aged, Historical, and Day 1 be separated, or do they share one inseparable Next Actions engine? | **Critical** (B, G) | Determines whether we are pricing displacement or the fallback. The most consequential single finding in Phase 0. |

**Exit criteria:** we can build and deploy ProWeb ourselves; we have an object-level inventory; we know what our changes can break; we have an evidence-based database target recommendation with costs; we know whether the product lines separate; 2.1 is priceable.

### Phase 1 — Make change safe and clear the 2027 wall (target 4–7 months, largely parallel)

| # | Task | Criticality | Notes |
|---|---|---|---|
| 1.1 | Consolidate source control to Azure DevOps; branch policies; **revoke direct-to-production push for third-party developers** | **Critical** (B, X) | 1.2 depends on it. Offshore third parties pushing straight to production is a named cyber gap. Resolve the TFS-vs-Azure-DevOps contradiction (§8) first. |
| 1.2 | Reproducible build → CI → automated deploy to lower environments | **Critical** (B) | 1.3 cannot run without somewhere to run it. Build reproducibility precedes pipeline automation. |
| 1.3 | **Characterization / golden-master test harness over the stored procedure surface** | **Critical** (B) | **Gates all of Phase 2.** The single most critical task in this roadmap — nothing downstream is safe without it. Sized off 0.2 and 0.3. |
| 1.4 | Masked or synthetic lower environment | **Critical** (B, X) | Blocks our own offshore delivery model, and without it offshore development means offshore PHI. Does not exist today; unsized by EY-P. |
| 1.5 | .NET Framework 4.5 → 4.8.1 | Non-critical | **Reclassified by the path choice.** Under the in-place fallback this was a prerequisite to .NET Core. Under displacement, new services are built on modern .NET and ProWeb is being retired, so this is stabilization of a system on its way out. Still worth doing — 4.5 has been unsupported since 2022 — but it blocks nothing and has no date. |
| 1.6 | **Windows Server 2016 → 2022** | **Critical** (D, X) | Hard deadline 12 Jan 2027, ~5.5 months out. Procure ESU as insurance now regardless of plan confidence. |
| 1.7 | Dependency remediation — iTextSharp CVE-2021-43113 (CVSS 9.8) and CVE-2017-9096 (8.8) first, then jQuery UI / Knockout / CPython | **Critical** (X) | An active critical vulnerability on an internet-facing application holding ~1m PHI records. Cheap to close; no reason to carry it. |
| 1.8 | WAF in front of ProWeb; DAST in the pipeline | **Critical** (X) | ProWeb is internet-facing with no WAF. EY-P prices the WAF at <$5k. Monitor → tune → enforce. |
| 1.9 | Observability baseline — app and DB telemetry into a single monitoring plane | Non-critical | Displacement is possible without a SIEM, and no fixed date applies. Ride it along with 1.2 — it supplies the change-impact signal Phase 2 benefits from, and the fragmented-logging gap is real but not acute. |
| 1.10a | **Database target decision, then in-place upgrade to SQL Server 2022** | **Critical** (D) | Deadline 12 Oct 2027. Decision gated on 0.11. Run Data Migration Assistant first; pin compatibility level and raise it deliberately behind 1.3. |
| 1.10b | Cloud migration (Managed Instance or Azure SQL Database) | Non-critical | Deferrable, and should be deferred. Revisit after the first product lines are displaced, when the architecture actually suits PaaS. If it proceeds: Azure DMS online mode or transactional replication, **not BACPAC**; vCore for Azure Hybrid Benefit; scale target resources for cutover. |

**Exit criteria:** we can change ProWeb and know within minutes if we broke it. Both 2027 platform floors cleared or formally deferred with ESU.

### Phase 2A — Divert the Flow: displace the aggregator first

Runs as soon as 1.3 and 0.9 allow. This is the opening displacement move because it is what unfreezes the schema for everything after it.

| # | Task | Criticality |
|---|---|---|
| 2A.1 | Stand up a decoupled analytical store fed by CDC from ProWeb, plus **Revert to Source** feeds for anything ProWeb only passes through (per 0.12) | **Critical** (B) — everything else in 2A depends on it |
| 2A.2 | Rebuild reports **iteratively, one at a time, to production** — not as a big-bang reporting migration | **Critical** (B) — scope set by 0.9 |
| 2A.3 | **Parallel running with reconciliation** against the existing Power BI outputs, with alerting on divergence | **Critical** (X) — cutting over on unverified numbers breaks the business's reporting |
| 2A.4 | Hunt down **"off system" workarounds** — manual spreadsheet manipulation between ProWeb output and what leadership actually sees | **Critical** (B) — skip it and 2A.3 produces mismatches nobody can explain, which stalls cutover |
| 2A.5 | Stage cutover by user cohort; **retire the direct-to-OLTP query path per report** | **Critical** (B) — this *is* the exit criterion. Without retiring the direct path the schema stays frozen and 2A delivered nothing structural |
| 2A.6 | Fix data quality issues **upstream**, not by reimplementing legacy workarounds in the new store | Non-critical — ongoing improvement; treat each case as a defect ticket against the upstream system |

**Deliberately not feature parity.** Per 0.9, decompose which outputs have real users. A smaller set of targeted reports is the goal; rebuilding every bloated report is the trap.

**Expect the numbers to disagree.** Legacy reports commonly contain undiscovered bugs, so new outputs rarely match. Build worked examples with known inputs and agreed expected outputs *with the business* before cutover, so we can tell which system is right rather than assuming legacy is.

**Exit criteria:** no operator-facing report queries the transactional database directly. From here, schema change is ours to make.

### Phase 2B — Extract Product Lines: displace by solution line

Gated on 0.13 confirming separability.

**Sequence: Discovery as a funded pilot, then Aged.** The pattern's own heuristic says take the *second riskiest* line, which would point at Aged or Historical, not Discovery. We are deviating deliberately and for a stated reason: with no test harness, no specification, and no internal engineering staff, the first slice is buying knowledge, not revenue. Discovery is the thinnest viable slice — 2% of volume, advisory-only, client retains billing and AR, and per p.17 it activates only ARC detection and coverage discovery.

The heuristic's real warning still binds: a 2% win will not sustain executive sponsorship across a multi-year programme. So **Aged is named upfront as the first real product line**, and Discovery is budgeted explicitly as pattern-establishing work — scaffolding, harness, deployment path, team shape — not as value delivery. If we cannot say what Discovery produced that makes Aged cheaper, the pilot failed.

| # | Task | Criticality |
|---|---|---|
| 2B.1 | **Discovery (pilot).** New service implementation; **Event Interception** on ISO/Verisk webhooks and SFTP arrivals; **Revert to Source** for placement files. Deliverable includes the reusable scaffolding, not just the feature | **Critical** (B) — but critical *by choice*, not by dependency. Nothing technically requires a pilot; we chose one because there is no harness or specification to learn from. The scaffolding it produces is what later lines depend on |
| 2B.2 | **Legacy Mimic** layer so clients, clearinghouses, and ISO/Verisk cannot tell which system is serving them | **Critical** (B, X) — no product line can cut over without it, and partner-visible breakage is contractual |
| 2B.3 | **Aged** — first real line, 32% of volume, 20–30% of collections. Staged cutover: beta cohort, then 1% / 5% / 10% | **Critical** (G) — the first genuine value delivery and what sustains sponsorship for 2B.4–5 |
| 2B.4 | **Historical** — 25% | Non-critical — deferrable; nothing depends on it |
| 2B.5 | **Day 1** — 42%, full lifecycle ownership, last | **Critical** (B) — ProWeb cannot be retired while Day 1 still runs on it, so the programme cannot complete without it. Deferrable in sequence, not in scope |
| 2B.6 | Progressive retirement of displaced ProWeb modules, contracted as named deliverables | **Critical** (B) — this is what distinguishes displacement from running two platforms |
| 2B.7 | **Removal of transitional architecture** — façade, CDC bridges, reconciliation jobs | **Critical** (B) — skipping it is the R9 failure. Fund and schedule it; do not leave it to goodwill |

Per Extract Product Lines: identify shared capabilities early and **value use over reuse**. Limit what the new product lines share, or we rebuild the over-generic system we are trying to escape.

### Phase 3 — Automation and AI enablement

Runs in parallel throughout, because it does not depend on ProWeb. **Almost everything here is non-critical by the §4 test and simultaneously where the client's return lives** — see the caveat in §4.

| # | Task | Criticality |
|---|---|---|
| 3.1 | Triage the **60% unplanned** automation opportunity by ProWeb dependency and by automation *class* (see R8) | **Critical** (G) — cheap, and it is what stops us underwriting the ~$620k figure before we have tested it |
| 3.2 | Ship off-platform automation now — the Python/Groq pipeline needs no ProWeb change | **Critical** (G) — the only value we can deliver early, and what buys credibility for the long items |
| 3.3 | Intelligent work routing; expected pay modelling; variance detection | Non-critical — high value, no dependency |
| 3.4 | Agentic task execution — Tort Recovery, PHI-minimized, human oversight, bounded action library | Non-critical — and deliberately late; see R-3.4 |
| 3.5 | Payer portal and telephony automation | Non-critical — **and we should resist it.** Highest-risk work in the roadmap (R8), separate risk class, separate estimate |
| 3.6 | Point the AI/ML pipeline at source data rather than ProWeb SQL where 0.12 shows ProWeb is only passing data through | Non-critical |

### Phase 4 — Data platform maturity

Reduced, because reporting displacement moved to 2A. Both items are non-critical by design — this is what deferrable work actually looks like.

| # | Task | Criticality |
|---|---|---|
| 4.1 | Extend the 2A analytical store into longitudinal claim analytics | Non-critical |
| 4.2 | Feature store for the ML pipeline | Non-critical |

### Fallback — in-place refactor (Path A)

If 0.13 shows the product lines cannot be separated, we fall back to extracting logic domain-by-domain inside ProWeb behind the 1.3 harness. Phase 2A still comes first and still applies. This path is slower to value, commits us closer to feature parity, and should be priced per domain only.

| Task | Criticality |
|---|---|
| Domain-by-domain logic extraction behind the 1.3 harness | **Critical** (B) |
| Trigger elimination — convert implicit DML side effects to explicit service calls | **Critical** (B, X) — must precede extraction of any dependent procedure |
| WCF → REST replacement (insurance and fax workflows) | **Critical** (B) — blocks the .NET Core step, and needs third-party partner coordination |
| .NET 4.8.1 → current .NET | **Critical** (B) — and **1.5 becomes critical again on this path** |
| Frontend: incremental React islands replacing Knockout/Kendo | Non-critical |

---

## 6. What can actually slip

The non-critical list, gathered. This is the answer to "what do we drop if Phase 0 comes back worse than expected."

| Task | Why it can wait | What we lose by waiting |
|---|---|---|
| 0.5 DR failover test | Blocks nothing, no date | Operating on unvalidated recovery targets. Cheap enough that deferring it is a false economy |
| 0.8 Vendor / license review | Park Road's legal workstream, not ours | Nothing, provided it is actually routed there |
| 1.5 .NET 4.5 → 4.8.1 | Displacement builds new services on modern .NET; ProWeb is being retired | Continued unsupported framework on a shrinking surface. **Becomes critical on the fallback path** |
| 1.9 Observability baseline | Displacement works without a SIEM | Weaker change-impact signal during 2A/2B; fragmented logging gap persists |
| 1.10b Cloud migration | In-place SQL 2022 clears the deadline | Nothing near-term. Deferring is the recommendation, not a concession |
| 2A.6 Upstream data quality | Ongoing rather than gating | Special cases accumulate in transformation logic |
| 2B.4 Historical | No dependency on it | 25% of volume stays on ProWeb longer |
| 3.3 / 3.4 / 3.6 | No ProWeb dependency | Real value deferred — this is the trade-off to make consciously |
| 3.5 Payer portals / telephony | No dependency, highest risk | The largest slice of the automation case. Resist until 3.1 has re-baselined it |
| 4.1 / 4.2 | Deferrable by design | Analytics maturity, not capability |
| Fallback: frontend React islands | Cosmetic relative to the rest | UI stays legacy |

**Two reclassifications worth flagging**, because they are consequences of the path choice rather than judgments about the work:

- **1.5 (.NET 4.8.1)** was a prerequisite when the plan was in-place refactoring, since .NET Core was the destination. Under displacement it is stabilization of a system being retired. If 0.13 forces the fallback, it becomes critical again.
- **1.10b (cloud migration)** is non-critical because 1.10a clears the deadline in place. EY-P presents these as alternatives; separating them is what makes the deadline affordable.

---

## 7. Risk register by task

Scored L(ikelihood) × I(mpact), H/M/L.

### Phase 0

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 0.1 Build validation | Repo is incomplete — missing config, undocumented manual deploy steps, binaries not in source control, a build that only works on one contractor's machine | **H×H** | Any "ask Droit for that file" during the attempt | Do this in week 1. A failed clean-room build is a go/no-go input, not a defect to fix later. Budget for reconstruction. |
| 0.2 Dependency map | Tooling cannot resolve dynamic SQL. Logic assembled at runtime in strings is invisible to static analysis | **H×M** | High `EXEC(@sql)` / `sp_executesql` density | Combine static analysis with runtime query capture from 0.3. Report coverage as a percentage — never imply the map is complete. |
| 0.3 Behavioral baseline | Capture window misses month-end, quarter-end, and payer-cycle edge cases; harness is built on a partial picture | **H×M** | Coverage gaps against the p.18 workflow map | Capture must span at least one full month-end. Start day 1 — calendar time we cannot compress later. |
| 0.4 Asset inventory | Still not provided, or SQL Server turns out older than 2017 / a lower edition than assumed | M×M | Repeated deferral, as happened with EY | Escalate as a close condition. If SQL is pre-2017, 1.10a timeline and cloud options both change. |
| 0.5 DR test | The test fails, or is refused because nobody will authorize a live failover | **M×H** | Reluctance to schedule a window | If a real failover is refused, that refusal is itself the finding — report it to Park Road. Do not accept untested RPO/RTO as fact. |
| 0.6 PHI / BAA | No BAA with Groq covering medical records; offshore access to production PHI is unpapered | **M×H** | Vague answers on where inference runs | Legal review before we touch anything. A BAA gap is a pre-close issue for Park Road, not our remediation item. |
| 0.7 Segregation RCA | Root cause is record identity, not the SFTP script — implying a schema change across the core data model | **M×H** | Account keys not client-scoped; collisions possible by construction | Size both options (compensating controls vs. identity re-model). Feeds 0.13 and 2B sequencing. |
| 0.8 Licenses | Kendo UI or another commercial component has a change-of-control or per-seat restriction that bites post-close | M×M | Missing license documentation in the VDR | Route to Park Road's legal workstream now, while it is still their cost. |
| 0.9 Reporting map | Report owners cannot be identified, so we cannot tell real outputs from bloat and end up rebuilding everything | **H×M** | Reports with no named owner; Power BI datasets querying tables directly | Named owner per report is the deliverable. Unowned reports are candidates for retirement, not migration. |
| 0.10 Vendor transition | Droit reads the takeover as displacement and disengages, taking the only system knowledge with it | **M×H** | Slow responses, gatekeeping on repo access | Design them in, not out — paid knowledge-transfer scope with named deliverables. Do not let commercial tension destroy the only map of the system. |
| 0.11 Azure Migrate | Output treated as a recommendation to follow rather than an input to weigh. Azure Migrate optimizes for a supported Azure landing zone; it has no view on whether moving *now* is wise given the displacement ahead | **M×M** | A target selected on tool output alone | Assessment answers *which targets are viable and at what cost*. Whether and when to move stays our judgment. Record both answers separately. |
| 0.11 | Assessment misses dependencies because discovery does not reach everything — jobs on unmanaged servers, ad-hoc linked servers, undocumented ETL | **M×H** | Discovery scope narrower than the 0.4 inventory, which may itself be incomplete | Reconcile 0.11 output line-by-line against 0.2 and 0.4. Any gap is unpriced migration risk; report it as such. |
| 0.11 | Cost estimate built on current on-prem sizing, which reflects a database doing application work. The profile changes after displacement, so the estimate expires | M×M | Sizing taken from peak CPU on the current box | Treat the monthly estimate as a decision input with a stated shelf life. Re-run during 2B. |
| 0.12 Source mapping | Upstream tracing stalls because the people who built the integrations have left, and Droit's knowledge only covers ProWeb inward | **M×M** | Nobody can explain how a feed arrives | Switch to downstream tracing from the integration point forward — the article's recommended fallback when legacy knowledge is lost. |
| 0.12 | Source systems turn out not to be reliable or available enough to feed us directly, discovered after we design for it | M×H | No availability data on client SFTP endpoints or the ISO/Verisk feed | Check cross-functional constraints on each source before designing Revert to Source into a slice. |
| 0.13 Separability | The four lines share one inseparable Next Actions engine, so Extract Product Lines does not apply and we are on the fallback path | **M×H** | Same tables and same procedures serving all four lines with only flag-based variation | The single most consequential Phase 0 finding. Run it early enough that the answer arrives before we commit to a Phase 2 shape. |

### Phase 1

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 1.1 Source control | Migration loses history, or two sources of truth run in parallel and diverge | M×M | The TFS/ADO contradiction in §8 goes unresolved | Resolve current state first. Freeze, migrate, verify, cut over — no dual-write period. |
| 1.2 CI | Build automated but deploy still manual and business-coordinated, so release risk is unchanged | M×M | "We'll wire deploy up later" | Definition of done is deploy-to-lower-environment, not green build. |
| 1.3 **Test harness** | Coverage is thinner than believed. Team gains confidence to change things that the harness does not justify | **H×H** | Coverage measured in tests written rather than behaviors pinned | Coverage expressed per business workflow, not per procedure. No slice enters Phase 2 until its behaviors are pinned. **This is the risk that decides whether the programme succeeds.** |
| 1.4 Masked environment | Masking breaks referential integrity, so the environment cannot reproduce real defects and gets abandoned | **M×M** | Developers asking for a production restore | Deterministic, referentially consistent masking. Validate by reproducing three historical production defects. |
| 1.5 .NET 4.8.1 | Behavioral differences in serialization, TLS, or date handling surface weeks later | M×M | Sparse harness coverage on money and date paths | Sequence after 1.3. Prioritize harness coverage on financial calculations. |
| 1.6 **Windows Server 2022** | We miss 12 Jan 2027. ProWeb runs on an unsupported OS holding ~1m PHI records | **H×H** | ESU not procured by Nov 2026 | Procure ESU as insurance **now**, regardless of plan confidence. Test legacy .NET/IIS compatibility on 2022 early — that is where an old stack surprises you. |
| 1.7 iTextSharp | The upgrade path is not drop-in — iText's licensing changed across major versions (AGPL/commercial) | **M×M** | Version pinned very old for a reason | Check licensing implications alongside the CVE fix. Consider replacing the component rather than upgrading it. |
| 1.8 WAF | Blocking mode breaks legitimate claim submission traffic; gets disabled under operational pressure | **M×M** | No monitor-mode learning period | Monitor → tune → enforce. Never straight to blocking on a revenue-path application. |
| 1.9 Observability | Telemetry added but nobody owns alerts; noise leads to it being ignored | M×M | No named owner | Tie to the vCISO / Director of Engineering hire. Tooling without ownership is theatre. |
| 1.10a Database target | **Azure SQL Database chosen and SQL Agent jobs, linked servers, or cross-database queries break late in migration.** Microsoft frames removing these as a benefit; here it is rework | **M×H** | A target named before 0.11 completes | Hard gate on 0.11. Managed Instance preserves instance-level features. Default to in-place SQL Server 2022. |
| 1.10a | In-place SQL Server 2022 upgrade surfaces deprecated T-SQL or compatibility-level behavior changes across thousands of procedures | **M×H** | No harness coverage on the affected procedures | Run Data Migration Assistant first. Pin compatibility level initially, raise it deliberately behind 1.3. |
| 1.10b | Cloud migration and displacement run concurrently, so two foundations move at once and neither rolls back cleanly | **M×H** | Cloud migration scheduled inside the Phase 2 window | Sequence, do not parallelize. This is why 1.10b is classified non-critical and deferred. |

### Phase 2A — Divert the Flow

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 2A.1 Analytical store | Fed exclusively by CDC from ProWeb, so it inherits ProWeb's data losses and batch timings and we have re-coupled to legacy in a new place | **M×H** | No Revert to Source feeds in the design | Use 0.12. Where ProWeb only passes data through, take it from source and gain the fields and timeliness ProWeb discards. |
| 2A.2 Report rebuild | Feature parity creeps in and we rebuild every bloated report, including ones with no users | **H×M** | Report count in scope equals report count in existence | 0.9's named-owner list is the scope gate. Unowned means retired. |
| 2A.3 Reconciliation | New outputs do not match legacy, and the mismatch is read as our defect when it is a legacy bug | **H×M** | No agreed expected-output examples before cutover | Build worked examples with known inputs and business-agreed outputs *before* parallel running, so we can attribute divergence rather than argue about it. |
| 2A.4 Off-system workarounds | Leadership's actual numbers come from manual spreadsheet steps nobody documented, so the new reports are "wrong" in ways we cannot reproduce | **M×H** | Anyone spending days per month producing a report the system supposedly produces | Ask directly and early, without blame. The article's experience is that nobody volunteers this because nobody wants to tell leadership the reporting does not work. |
| 2A.5 Cutover | Reports retired before every consumer is found; a downstream spreadsheet or client deliverable breaks | **M×M** | Consumers identified by system rather than by person | Cohort-based cutover with a rollback window per report. |
| 2A.6 Upstream fixes | Data quality issues get patched in the new store because fixing upstream is slower, recreating legacy workarounds in new code | **M×M** | Transformation logic accumulating special cases | Treat every new special case as a defect ticket against the upstream system. |
| 2A | **The phase is deprioritized** as "just reporting" against louder platform work, so the schema stays frozen and 2B stalls | **M×H** | 2A slipping while 2B slices get scheduled | Frame 2A internally and to the client as the schema unlock, not as an analytics project. Nothing in 2B is safe until it lands. |

### Phase 2B — Extract Product Lines

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 2B.1 Discovery pilot | Delivers a feature but no reusable scaffolding, so Aged re-learns everything and the pilot's justification evaporates | **M×H** | Pilot scoped as a feature rather than as scaffolding | Contract the scaffolding as the named deliverable. Acceptance test: state what makes Aged cheaper. |
| 2B.1 | A 2% slice fails to hold executive attention, funding softens before Aged starts | **M×H** | Sponsor engagement dropping after the pilot demo | Name Aged upfront with a date. This is the known cost of deviating from the second-riskiest heuristic — manage it, do not ignore it. |
| 2B.2 **Legacy Mimic** | The mimic layer is incomplete and a partner integration notices the change — ISO/Verisk, a clearinghouse, or a client's SFTP expectations | **M×H** | Partner-facing contracts not inventoried before the first cutover | Inventory external consumers in 0.12. Partner-visible behavior changes are calendar-driven and outside our control. |
| 2B.2 | Mimic becomes permanent because it is easier to keep than to retire, quietly becoming the new legacy | **M×M** | No retirement date attached at build time | Transitional Architecture discipline: every mimic component gets a removal owner and date when it is created. |
| 2B.3 Aged cutover | Staged traffic shift lacks a genuine rollback path, so 10% becomes all-or-nothing under pressure | **M×H** | No tested rollback at the 1% stage | Prove rollback at 1% before going to 5%. The gradient only works if each step is reversible. |
| 2B.3 | Aged is where recovery economics live (20–30% of collections). A displacement defect here has direct revenue consequence for the client | **M×H** | Harness coverage on Aged workflows below Discovery's | Do not start Aged until its behaviors are pinned to at least the standard Discovery reached. |
| 2B.4–5 | Shared capabilities accumulate between lines, and we rebuild the over-generic system we were escaping | **H×M** | A shared services layer growing with each new line | Value use over reuse. Duplication between product lines is the intended trade-off, not a defect. |
| 2B.6–7 | **Displacement stalls part-way. We run two platforms indefinitely** — the classic failure, worse than either endpoint | **M×H** | Retirement milestones slipping while new-build continues | Contract module retirement as named deliverables. No new line starts until the prior module is dark. Fund transitional-architecture removal explicitly. |
| 2B | Segregation defect (0.7) is inherited into the new services because record identity was copied rather than re-modelled | **M×H** | New services reusing ProWeb account keys unchanged | This is the opportunity to fix it properly. Displacement is when identity can change; refactoring in place is when it cannot. |

### Phase 3 — Automation and AI

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 3.1 Triage | The unplanned 60% is assumed to have the same unit economics as the planned 40% | **H×M** | A single blended savings figure | Re-baseline the ~$620k / ~$4.61-per-claim opportunity ourselves before committing. |
| 3.2 Off-platform | Groq dependency: single provider, hosted models (Llama 4 Scout, gpt-oss-120b) that can be deprecated or repriced | **M×M** | No provider abstraction in the Python codebase | Abstract the inference interface. Cheap now, and it protects margin. |
| 3.3 Routing / expected pay | Models trained on historical outcomes encode current process bias; measured lift fails to appear | M×M | No holdout or champion/challenger design | Insist on holdout groups. Do not let "AI-driven" substitute for measured lift. |
| R-3.4 Agentic execution | An autonomous action creates a billing or compliance error at scale before anyone notices | **M×H** | Action library expanding faster than the audit trail | Bounded action library, action-level audit, reversibility, staged rollout by volume. PHI-minimized scoping is right — keep it. Sequence late deliberately. |
| 3.5 **Payer portals / telephony** | The bulk of remaining manual cost is portal RPA plus voice — brittle, breaks on every payer UI change, carries per-payer maintenance forever | **H×H** | Estimated as "workflow automation" alongside deterministic work | Separate estimate, separate risk class, separate commercial treatment. Maintenance is the real cost. Consider clearinghouse/API routes before per-payer RPA. |
| 3.6 Revert to source | Repointing the ML pipeline changes its input distribution, and model performance shifts without anyone attributing it to the plumbing change | **M×H** | No parallel scoring during the switch | Score both feeds in parallel and compare before cutting over. Model drift caused by an infrastructure change is very hard to diagnose after the fact. |

### Fallback path (in-place refactor)

| Task | What goes wrong | L×I | Mitigation |
|---|---|---|---|
| Logic extraction | Effort lands at a multiple of EY-P's 1,600–2,200 hours; we are anchored to a number we did not produce | **H×H** | Price per domain after 0.2. Contract per domain. Never quote EY-P's figure as ours. |
| **Triggers** | Invisible DML side effects. New service-layer writes bypass or double-fire them, corrupting claim state | **H×H** | **Top technical hazard, above stored procedures.** Map fan-out exhaustively; convert triggers to explicit calls before extracting dependent procedures. Silent corruption in a billing system is the worst available failure mode. |
| WCF | No .NET Core equivalent; CoreWCF covers part of the surface. Insurance and fax integrations are third-party contracts we cannot change unilaterally | **H×H** | Inventory every endpoint and consumer in Phase 0. Partner coordination is calendar-driven and outside our control. |
| .NET Core | Hard-blocked on WCF, plus System.Web-coupled Razor views and the ADO connector | **H×M** | Communicate the dependency chain. EY-P presents .NET Core as "also being considered" without noting it is gated. |
| Frontend | React islands beside Knockout/Kendo produce two state models and a worse UX than either | **M×M** | Migrate by whole screen, never by widget within a screen. |
| Parallel work | Extraction runs alongside contractor feature work and the two collide continuously | **H×M** | Domain ownership boundaries and a change freeze per domain under extraction. |

---

## 8. Top risks, ranked

Ranked by expected cost to Dualboot, not by technical interest.

| # | Risk | Why it ranks here |
|---|---|---|
| **R1** | We anchor commercially to EY-P's 1,600–2,200h estimate | Direct margin exposure on the largest item. Self-inflicted and entirely avoidable. |
| **R2** | Change begins before the characterization harness is real | Silent data corruption in a claims platform serving ~100 customers. Existential to the engagement. |
| **R3** | Clean-room build fails / repo is incomplete | Discovered in week 1 if we look, month 6 if we do not. Cheap test, enormous information value. |
| **R4** | Product lines do not separate (0.13), forcing the fallback path | Collapses the whole Phase 2 shape and puts triggers back at the centre of the programme. |
| **R5** | Droit disengages and system knowledge is lost | The only specification of the system is in contractors' heads. |
| **R6** | Windows Server 2016 (12 Jan 2027) and SQL Server 2017 (12 Oct 2027) expire mid-programme | Compliance and insurance exposure with ~1m PHI records. EY-P rated this Low. |
| **R7** | Cross-client segregation defect recurs on our watch | Fourth occurrence, attributable to us as operator. Reputationally worse than any technical slip. |
| **R8** | The remaining 60% automation opportunity is portal/telephony RPA | Where the promised $620k lives, and the hardest class of work to sustain. |
| **R9** | Displacement stalls part-way and transitional architecture becomes permanent | Two platforms forever. The signature failure of this approach, and it arrives quietly. |
| **R10** | Phase 2A is deprioritized as "just reporting", so the schema never unfreezes | Blocks 2B without ever appearing as a blocker on a plan. |
| **R11** | A database target is committed before the Azure Migrate assessment, or cloud migration runs concurrently with displacement | Azure SQL Database removes exactly the instance-level features a T-SQL platform runs on. Two moving foundations removes rollback. |
| **R12** | Granite hires a Director of Engineering who re-litigates our scope | Structural commercial risk. Anticipate it; ideally influence the hire. |
| **R13** | Offshore PHI exposure without a masked environment | Blocks our own delivery model until 1.4 lands. |

---

## 9. Contradictions in the source report to resolve

The report is a working draft marked "PRELIMINARY DRAFT FOR REVIEW" throughout, and it does not fully reconcile with itself. Each of these changes our scope:

- **Source control:** p.25 says code is on **TFS** with Azure DevOps "evaluated but not initiated." p.44 says Granite "leverages **Azure DevOps** for source control." p.36 lists TFS, Azure Repos/Git, *and* Azure Repos/TFVC. `[ASSUMPTION]` the Python AI/ML repo is on Azure DevOps while ProWeb remains on TFS — confirm, since it determines whether 1.1 is a migration or a consolidation.
- **Inference runtime:** p.31 refers to "**Ollama**-based open source models" while the same page's diagram and p.32 specify **GroqCloud** with Llama 4 Scout and gpt-oss-120b. Ollama is a local runtime; Groq is hosted. Either there is local inference infrastructure nobody described, or this is a drafting error.
- **"Azure SQL"** used without specifying Database vs. Managed Instance (p.25). See §2.5.
- **R&D as % of revenue:** ~9% on pp.7 and 9, ~8% on p.15.
- **Incident count:** management said five, documents showed seven. Sets a baseline for how to weight unverified management statements generally.
- **SQL Server version:** inferred, never confirmed.
- p.26 footnote markers do not match the footnote list. Minor, but consistent with a document whose numbers may still move.

---

## 10. What we need before we price anything

1. Stored procedure and trigger inventory with complexity metrics (0.2)
2. Successful clean-room build (0.1)
3. **Product line separability answer (0.13)** — determines which Phase 2 we are pricing
4. Confirmed SQL Server version, edition, licensing (0.4)
5. Azure Migrate assessment output (0.11)
6. Reporting owner list and real-vs-bloat split (0.9)
7. Data flow map to ultimate source (0.12)
8. Segregation defect root cause (0.7)
9. Droit transition terms (0.10)
10. Close date — everything in the 2027 analysis is relative to it

Every item on this list is a Critical task in Phase 0. That is not a coincidence — Phase 0's criticality and our pricing dependencies are the same list.

**Commercial posture:** T&M or capped-T&M for Phase 0, Phase 1, and the Discovery pilot. Fixed price only per product line, and only after 0.2 and 0.13. We should be willing to walk away from a fixed-price commitment on full logic extraction — and say so early rather than discovering our position under pressure later.

---

## 11. Open questions

- Is our role remediation delivery, modernization execution, post-close R&D capacity, or a second-opinion read for Park Road? `[ASSUMPTION]` this note assumes delivery takeover. **If it is value delivery rather than modernization, the criticality ordering in §5 inverts and Phase 3 leads.**
- Do we displace Droit/Lapiz, sit alongside them, or absorb them?
- What is the expected hold period? A 2B sequence running Discovery → Aged → Historical → Day 1 needs a horizon we do not know.
- Does Park Road expect the ~$620k automation savings in the model? If underwritten, R8 becomes their risk and we should say so before close.
- Who owns the fourth segregation incident if it happens after we take over? Answer this contractually.
- Is there a BAA with Groq covering PHI in medical records?
- Does Park Road have an existing Azure footprint or enterprise agreement? Changes the 0.11 cost case.
- Would Park Road support us influencing the Director of Engineering hire?

---

## Verification note

Microsoft lifecycle dates in §2.3 and R6 verified against Microsoft Learn on 29 Jul 2026: [SQL Server 2017](https://learn.microsoft.com/en-us/lifecycle/products/sql-server-2017) extended end 2027-10-12; [Windows Server 2016](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2016) extended end 2027-01-12. Azure migration guidance in §2.5, 0.11, and 1.10 from [Migration overview: SQL Server to Azure SQL Database](https://learn.microsoft.com/en-us/data-migration/sql-server/database/overview) — that page covers **Azure SQL Database only** and does not address Managed Instance, so the Database-vs-MI comparison is reasoned rather than sourced. Displacement patterns in §2.6, §3, and Phase 2 verified against the source article; four pattern pages read in full (Critical Aggregator, Divert the Flow, Extract Product Lines, Revert to Source), the other four summarized from cross-references — see [[patterns-of-legacy-displacement]] for status and for the correction log on that note. Criticality classifications in §5 and §6 are ours, derived from the dependency structure of this roadmap rather than from any source document. All Granite-specific figures trace to the EY-P report at the page numbers cited in [[park-road]]. Every inference is tagged `[ASSUMPTION]`; no technical detail about Granite's environment has been supplied where the report is silent.
