---
type: Note
status: Draft
related_to: "[[park-road]]"
---

# Park Road / Granite — Platform Takeover Roadmap & Risk Register

**Audience:** Dualboot internal. Candid about delivery risk to us.
**Premise:** post-close, Dualboot takes over ProWeb as the delivery partner.
**Source of record:** EY-P Project Granite TDD, 12 May 2026 (working draft). See [[park-road]].
**Framing decision:** rebuild vs. modernize-in-place is unresolved in diligence, so this roadmap routes both through a Phase 2 decision gate rather than assuming one.

---

## 1. The honest read

We would be taking over a platform where **the application is a passthrough and the product is a database**. All business logic — claim state, Next Actions, payer routing, coverage rules, eligibility — lives in T-SQL stored procedures and triggers. There is no automated test suite, no CI/CD, no reproducible build proof, no specification, and no internal engineering staff. The people who understand the system are contractors at a vendor (Droit) that also acts as final code controller.

That combination is the whole risk story. Every technical task below is harder than its label because we cannot yet answer the question *"if I change this, what breaks?"* — and neither can anyone else.

The good news is real: the Python AI/ML stack (BioBERT, XGBoost, Groq-hosted LLMs) is architecturally separate from ProWeb, well-maintained, and can be extended without touching the monolith. That is where early value lives and where we should want to be measured first.

---

## 2. Where we disagree with EY-P

This is the substance of our point of view. EY-P's report is competent, but four things in it would hurt us if we signed to them.

**2.1 The business logic refactor estimate is not credible as a commitment.**
EY-P sizes business logic refactoring at **1,600–2,200 hours / ~$140k–$190k** — roughly one person-year — derived from "contractor scrum team effort." That is to extract the operating logic of a platform running ~$9.5m of revenue, ~100 customers, and the full ARC claim lifecycle out of the database. We should treat that figure as a **floor, not an estimate**, and refuse to price it until Phase 0 produces a stored-procedure inventory (object count, lines of T-SQL, cyclomatic complexity, trigger fan-out). `[ASSUMPTION]` Our expectation is a materially larger number; we should range-check against internal comparables rather than inventing one here.

**2.2 EY-P mis-sequenced the enabling investment.**
CI/CD and automated testing sit in EY-P's "Process & SDLC enhancements" bucket at **$15k–$25k each**, described as improvements that "reduce delivery friction." They are not enhancements. **A characterization test harness is a hard prerequisite of the XL refactor.** You cannot safely move logic out of stored procedures with no behavioral oracle to prove equivalence. Any plan that runs the refactor before the harness is a plan to break claims processing for 100 customers. This reordering is the single most important correction we make to their roadmap.

**2.3 The Windows Server 2016 deadline is much closer than "Low priority / phased."**
EY-P rates the OS upgrade **Low** priority with a "~3–12 months (phased)" timeline and leaves it unsized. Verified against Microsoft lifecycle data: **Windows Server 2016 extended support ends 12 January 2027** and **SQL Server 2017 extended support ends 12 October 2027**. As of today (29 Jul 2026) that is **~5.5 months** and **~14.5 months**. EY-P's tech debt plan is **22–28 months**. Both platform floors go unsupported well before the modernization they are supposed to survive. For a business holding ~1m PHI records, running unsupported OS and DB is a HIPAA and cyber-insurance argument, not just a patching inconvenience. This is a compound 2027 wall and it should be Phase 1, not "phased, low."

**2.4 The recurring data segregation failure is unsized and is probably a schema problem.**
Cross-client data exposure has now happened **three times** — 2022, June 2024 (571 non-client accounts), January 2026 (174 patients' SSN/DOB/contact plus 11 misattached medical records). EY-P's own description of the 2026 event names the cause: *"account numbering collisions and SFTP processing logic."* Account identity is not reliably client-scoped. That is a data model defect touching record identity, not an SFTP script bug — which is why patching it twice did not hold. Nobody has sized fixing it, and the regulatory and contractual exposure follows whoever operates the platform. **If we take over operations, we inherit the fourth occurrence.**

---

## 3. Proposed roadmap

### Phase 0 — Establish control and buy information (target 4–6 weeks, ideally starts pre-close)

Cheap, fast, and it converts most of our unpriced risk into priced risk. We should push hard to run as much of this as access allows *before* signing a delivery scope.

| # | Task | Why it exists |
|---|---|---|
| 0.1 | Clean-room build validation — clone the repo, build ProWeb from scratch, deploy to a throwaway environment | With no CI, nobody has proof the repository is complete and buildable. This is the single highest-information test available to us. |
| 0.2 | Static dependency map — every stored procedure, trigger, view, SQL Agent job, linked server, and cross-database reference | Produces the object inventory that lets us size 2.1 honestly, and exposes the trigger fan-out. |
| 0.3 | Behavioral baseline capture — production query/workflow logging to build the regression oracle | The characterization harness needs recorded real inputs and outputs. Start collecting on day 1; it is calendar-bound, not effort-bound. |
| 0.4 | Environment and asset inventory; confirm SQL Server version, edition, and licensing | EY requested an asset inventory and **never received one**. SQL Server 2017 is EY-P's inference, not a confirmed fact. |
| 0.5 | End-to-end DR failover test (Orlando → Atlanta) | Stated RPO/RTO of 15–30 min are design targets that have never been validated. |
| 0.6 | PHI data-flow map and BAA review — including Groq and offshore developer access | Medical records are sent to a third-party inference provider; offshore contractors push to production. Both need BAA and access-control answers. |
| 0.7 | Root cause analysis of the cross-client segregation defect | Determines whether this is a script fix or a tenancy/schema change. Changes the shape of the whole roadmap. |
| 0.8 | Vendor, license, and change-of-control review — Kendo UI/Telerik, Veeam, Datto, Kaseya, Mineral, KnowBe4, Coalition | Commercial licenses at change of control are a classic post-close surprise. |
| 0.9 | Reporting dependency map — which Power BI reports bind to which OLTP objects | Reporting runs directly off the transactional database, so schema changes break operators' reports. This map defines our refactor blast radius. |
| 0.10 | Vendor transition and knowledge-capture plan with Droit and Lapiz | Droit is long-tenured and is the final code controller. Their knowledge is the only specification that exists. |

**Exit criteria:** we can build and deploy ProWeb ourselves; we have an object-level inventory; we know what our changes can break; 2.1 is priceable.

### Phase 1 — Make change safe and clear the 2027 wall (target 4–7 months, largely parallel)

| # | Task | Notes |
|---|---|---|
| 1.1 | Consolidate source control to Azure DevOps; branch policies; **revoke direct-to-production push for third-party developers** | Governance fix and a named cyber gap. Resolve the TFS-vs-Azure-DevOps contradiction (§6) first. |
| 1.2 | Reproducible build → CI → automated deploy to lower environments | Sequenced deliberately: build reproducibility precedes pipeline automation. |
| 1.3 | **Characterization / golden-master test harness over the stored procedure surface** | The enabling investment. Gates everything in Phase 2A/2B. Sized off 0.2 and 0.3, not guessed. |
| 1.4 | Masked or synthetic lower environment | Prerequisite for offshore development without offshore PHI exposure. Does not exist today; unsized by EY-P. |
| 1.5 | .NET Framework 4.5 → 4.8.1 | EY-P's stabilization step. Low architectural value, real security value. |
| 1.6 | **Windows Server 2016 → 2022** | Hard deadline 12 Jan 2027. If we cannot make it, procure ESU and document mitigating controls. |
| 1.7 | Dependency remediation — iTextSharp CVE-2021-43113 (CVSS 9.8) and CVE-2017-9096 (8.8) first, then jQuery UI / Knockout / CPython | The two iTextSharp findings are the only critical/high items and are cheap to close. |
| 1.8 | WAF in front of ProWeb; DAST in the pipeline | ProWeb is internet-facing with ~1m PHI records and no WAF. EY-P prices the WAF at <$5k — take it immediately. |
| 1.9 | Observability baseline — app and DB telemetry into a single monitoring plane | No SIEM today. Also gives us the change-impact signal we need during refactoring. |
| 1.10 | SQL Server upgrade decision and execution (in-place 2022 vs. Azure SQL MI) | Deadline 12 Oct 2027. Azure SQL **Database** likely breaks SQL Agent jobs, linked servers, and cross-DB queries — Managed Instance is the realistic cloud target. Depends on 0.2. |

**Exit criteria:** we can change ProWeb and know within minutes if we broke it. Both 2027 platform floors cleared or formally deferred with ESU.

### Phase 2 — Decision gate: in-place vs. strangler

Decide once, with Phase 0/1 evidence in hand. Inputs: stored-procedure inventory and complexity, trigger fan-out, reporting coupling, segregation-defect RCA outcome, hold-period horizon, and who Granite hires as Director of Engineering.

**Bias:** if 0.7 concludes the segregation defect requires a tenancy/identity change, that pushes hard toward Path B — you do not want to re-model record identity inside a monolith you cannot test.

#### Path A — In-place modernization

| # | Task |
|---|---|
| A1 | Domain-by-domain logic extraction from stored procedures into a .NET service layer, behind the 1.3 harness |
| A2 | Trigger elimination — convert implicit DML side effects into explicit service calls |
| A3 | WCF → REST replacement (insurance and fax workflows) |
| A4 | .NET 4.8.1 → current .NET, **blocked on A3** |
| A5 | Frontend: incremental React islands replacing Knockout/Kendo views |
| A6 | Database modernization completed per 1.10 |

*Profile:* lower disruption, longer tail, value arrives late, and we carry the monolith's constraints the whole way. Every step is reversible.

#### Path B — Strangler fig

| # | Task |
|---|---|
| B1 | API façade / gateway in front of ProWeb; all new traffic routed through it |
| B2 | New service layer and domain model — **start with Discovery** (2% of volume, advisory-only, client retains billing and AR: the lowest blast radius in the portfolio) |
| B3 | CDC / dual-write and read-model separation; ProWeb DB stays system of record initially |
| B4 | New UI shell; migrate workflow-by-workflow |
| B5 | Progressive retirement of ProWeb modules |

*Profile:* higher upfront cost, earlier demonstrable value, cleaner AI-native target, but dual-run complexity and a real risk of permanent half-migration.

### Phase 3 — Automation and AI enablement

| # | Task |
|---|---|
| 3.1 | Triage the **60% unplanned** automation opportunity by ProWeb dependency and by automation *class* (see R12) |
| 3.2 | Ship off-platform automation now — the Python/Groq pipeline needs no ProWeb change |
| 3.3 | Intelligent work routing; expected pay modelling; variance detection |
| 3.4 | Agentic task execution — Tort Recovery, PHI-minimized, human oversight, bounded action library |
| 3.5 | Payer portal and telephony automation — **treat as a separate risk class, not part of the same estimate** |

### Phase 4 — Data platform

| # | Task |
|---|---|
| 4.1 | Analytical separation via CDC into a warehouse |
| 4.2 | Migrate Power BI reporting off OLTP (unblocks refactor freedom retroactively) |
| 4.3 | Feature store for the ML pipeline |

EY-P declined to size the warehouse, calling it not feasible near-to-medium term. We should agree on sequence but disagree on framing: 4.2 is not an analytics nicety, it is what removes the reporting coupling that constrains Phase 2.

---

## 4. Risk register by task

Scored L(ikelihood) × I(mpact), H/M/L.

### Phase 0

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 0.1 Build validation | Repo is incomplete — missing config, undocumented manual deploy steps, binaries not in source control, a build that only works on one contractor's machine | **H×H** | Any "ask Droit for that file" during the attempt | Do this in week 1. A failed clean-room build is a go/no-go input, not a defect to fix later. Budget for reconstruction. |
| 0.2 Dependency map | Tooling cannot resolve dynamic SQL. Logic assembled at runtime in strings is invisible to static analysis | **H×M** | High `EXEC(@sql)` / `sp_executesql` density | Combine static analysis with runtime query capture from 0.3. Explicitly report coverage as a percentage — never imply the map is complete. |
| 0.3 Behavioral baseline | Capture window misses month-end, quarter-end, and payer-cycle edge cases; harness is built on a partial picture | **H×M** | Coverage gaps against the p.18 workflow map | Capture must span at least one full month-end. Start day 1 — this is calendar time we cannot compress later. |
| 0.4 Asset inventory | Still not provided, or SQL Server turns out older than 2017 / a lower edition than assumed | M×M | Repeated deferral, as happened with EY | Escalate as a close condition. If SQL is pre-2017, 1.10 timeline and cloud options both change. |
| 0.5 DR test | The test fails, or is refused because nobody will authorize a live failover | **M×H** | Reluctance to schedule a window | If a real failover is refused, that refusal is itself the finding — report it to Park Road. Do not accept untested RPO/RTO as fact. |
| 0.6 PHI / BAA | No BAA with Groq covering medical records; offshore access to production PHI is unpapered | **M×H** | Vague answers on where inference runs | Legal review before we touch anything. If a BAA gap exists it is a pre-close issue for Park Road, not our remediation item. |
| 0.7 Segregation RCA | Root cause is record identity, not the SFTP script — implying a schema change across the core data model | **M×H** | Account keys not client-scoped; collisions possible by construction | Size two remediation options (compensating controls vs. identity re-model) and let the gate in Phase 2 consume the answer. |
| 0.8 Licenses | Kendo UI or another commercial component has a change-of-control or per-seat restriction that bites post-close | M×M | Missing license documentation in the VDR | Route to Park Road's legal workstream now, while it is still their cost. |
| 0.9 Reporting map | Operators depend on reports nobody documented; we discover coupling by breaking it | **H×M** | Power BI datasets querying tables directly rather than views | Treat every OLTP object touched by a report as a frozen interface until Phase 4.2. |
| 0.10 Vendor transition | Droit reads the takeover as displacement and disengages, taking the only system knowledge with it | **M×H** | Slow responses, gatekeeping on repo access | Design them in, not out — paid knowledge-transfer scope with named deliverables. Do not let commercial tension destroy the only map of the system. |

### Phase 1

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 1.1 Source control | Migration loses history, or two sources of truth run in parallel and diverge | M×M | The TFS/ADO contradiction in §6 goes unresolved | Resolve current state first. Freeze, migrate, verify, cut over — no dual-write period. |
| 1.2 CI | Build automated but deploy still manual and business-coordinated, so release risk is unchanged | M×M | "We'll wire deploy up later" | Definition of done is deploy-to-lower-environment, not green build. |
| 1.3 **Test harness** | Coverage is thinner than believed. Team gains confidence to refactor that the harness does not justify | **H×H** | Coverage measured in tests written rather than behaviors pinned | Coverage expressed per business workflow, not per procedure. No domain enters Phase 2 extraction until its behaviors are pinned. **This is the risk that decides whether the programme succeeds.** |
| 1.4 Masked environment | Masking breaks referential integrity, so the environment cannot reproduce real defects and gets abandoned | **M×M** | Developers asking for a production restore | Deterministic, referentially consistent masking. Validate by reproducing three historical production defects. |
| 1.5 .NET 4.8.1 | Behavioral differences in serialization, TLS, or date handling surface in edge cases weeks later | M×M | Sparse harness coverage on money and date paths | Sequence after 1.3. Prioritize harness coverage on financial calculations. |
| 1.6 **Windows Server 2022** | We miss 12 Jan 2027. ProWeb runs on an unsupported OS holding ~1m PHI records | **H×H** | ESU not procured by Nov 2026 | Procure ESU as insurance **now**, regardless of plan confidence. Cheap relative to being unsupported. Legacy app compatibility on 2022 needs testing early — this is where an old .NET/IIS stack surprises you. |
| 1.7 iTextSharp | The upgrade path is not drop-in — iText's licensing changed across major versions (AGPL/commercial) | **M×M** | Version pinned very old for a reason | Check licensing implications alongside the CVE fix. Consider replacing the component rather than upgrading it. |
| 1.8 WAF | WAF in blocking mode breaks legitimate claim submission traffic; gets disabled under operational pressure | **M×M** | No monitor-mode learning period | Monitor mode → tune → enforce. Never straight to blocking on a revenue-path application. |
| 1.9 Observability | Telemetry added but nobody owns alerts; noise leads to it being ignored | M×M | No named owner | Tie to the vCISO / Director of Engineering hire. Tooling without ownership is theatre. |
| 1.10 SQL upgrade | Azure SQL Database chosen, then cross-database queries, SQL Agent jobs, or linked servers break late in migration | **M×H** | Target selected before 0.2 completes | Do not choose a target before the dependency map exists. Managed Instance is the realistic cloud option given database-centric design. Deadline 12 Oct 2027. |

### Phase 2 — Path A (in-place)

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| A1 Logic extraction | Effort lands at a multiple of EY-P's 1,600–2,200 hours; we are anchored to a number we did not produce | **H×H** | Any commercial commitment made before 0.2 | Price per domain after inventory. Contract per domain, not for the whole extraction. Never quote EY-P's number back as ours. |
| A1 | Extraction proceeds in parallel with contractor feature work, and the two collide continuously | **H×M** | Multiple teams on parallel code paths — EY-P already flags this as current practice | Domain ownership boundaries and a change freeze per domain under active extraction. |
| A2 **Triggers** | Triggers fire invisible side effects. New service-layer writes bypass or double-fire them, corrupting claim state | **H×H** | Trigger count and fan-out from 0.2 | **Treat triggers as the top technical hazard, above stored procedures.** Map exhaustively, convert to explicit calls before extracting the procedures that depend on them. Silent data corruption in a billing system is the worst failure mode available here. |
| A3 WCF | WCF has no .NET Core equivalent; CoreWCF covers only part of the surface. Insurance and fax integrations are third-party contracts we cannot unilaterally change | **H×H** | Partner-side change required | Inventory every WCF endpoint and its external consumer in Phase 0. Partner coordination is calendar-driven and outside our control — surface it as a schedule risk early. |
| A4 .NET Core | Hard-blocked on A3, plus System.Web-coupled Razor views and the ADO connector | **H×M** | A3 slipping | Communicate the dependency chain explicitly. EY-P presents .NET Core as "also being considered" without noting it is gated. |
| A5 Frontend | React islands coexisting with Knockout/Kendo produce two state models and a worse UX than either | **M×M** | Shared state across framework boundaries | Migrate by whole screen, never by widget within a screen. Accept a visibly mixed UI for a period. |
| A6 Database | Covered in 1.10 | — | — | — |

### Phase 2 — Path B (strangler)

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| B1 Façade | Façade becomes a second passthrough layer with logic still in the database — new architecture diagram, same problem | **M×H** | No logic actually moved after the first domain | Success criterion is logic relocated, not traffic proxied. |
| B2 First domain | Discovery is chosen for low blast radius but is too small to prove the pattern, so the second domain re-learns everything | M×M | First domain ships with no reusable scaffolding | Explicitly budget the first domain as pattern-establishing, and say so commercially. |
| B3 CDC / dual-write | Dual-write drift between old and new stores. Reconciliation becomes permanent operational cost | **H×H** | No automated reconciliation from day 1 | Single writer per entity, always. If dual-write is unavoidable, ship reconciliation with it, not after. |
| B4 UI migration | Operators work across two UIs for months; throughput drops and adoption resistance builds | **M×H** | Ops team not represented in sequencing | Sequence by operator journey rather than by technical convenience. Ops leadership co-owns the order. |
| B5 Retirement | Migration stalls at 60%. We run two platforms indefinitely — the classic strangler failure and worse than either endpoint | **M×H** | Retirement milestones slipping while new-build continues | Contract retirement of specific ProWeb modules as named deliverables. No new-domain start until the prior module is dark. |

### Phase 3 — Automation and AI

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 3.1 Triage | The unplanned 60% is assumed to have the same unit economics as the planned 40% | **H×M** | A single blended savings figure | Re-baseline the ~$620k / ~$4.61-per-claim opportunity ourselves before committing to it. |
| 3.2 Off-platform | Groq dependency: single provider, hosted models (Llama 4 Scout, gpt-oss-120b) that can be deprecated or repriced | **M×M** | No provider abstraction in the Python codebase | Abstract the inference interface. Cheap now, and it is also a margin-protection measure. |
| 3.3 Routing / expected pay | Models trained on historical outcomes encode current process bias; measured lift fails to appear | M×M | No holdout or champion/challenger design | Insist on holdout groups. Do not let "AI-driven" substitute for measured lift. |
| 3.4 Agentic execution | An autonomous action taken against a claim creates a billing or compliance error at scale before anyone notices | **M×H** | Action library expanding faster than the audit trail | Bounded action library, full action-level audit, reversibility, staged rollout by volume. PHI-minimized scoping is right — keep it. |
| 3.5 **Payer portals / telephony** | This is the bulk of the remaining manual cost, and it is portal RPA plus voice — brittle, breaks on every payer UI change, and carries per-payer maintenance forever | **H×H** | Estimated as "workflow automation" alongside deterministic work | Separate estimate, separate risk class, separate commercial treatment. Ongoing maintenance is the real cost, not the build. Consider clearinghouse/API routes before RPA per payer. |

### Phase 4 — Data platform

| Task | What goes wrong | L×I | Leading indicator | How we de-risk |
|---|---|---|---|---|
| 4.1 Warehouse | Built before reporting migrates, so it becomes a parallel unused asset | M×M | No report migration plan | Fund 4.1 and 4.2 as one workstream. |
| 4.2 Reporting migration | Operators resist losing familiar reports; OLTP coupling persists and keeps constraining Phase 2 | **M×H** | Report owners unidentified | Use the 0.9 map. Migrate report-by-report with the owner named. |

---

## 5. Top risks, ranked

Ranked by expected cost to Dualboot, not by technical interest.

| # | Risk | Why it tops the list |
|---|---|---|
| **R1** | We anchor commercially to EY-P's 1,600–2,200h business logic estimate | Direct margin exposure on the largest item in the programme. Purely self-inflicted and entirely avoidable. |
| **R2** | Refactoring begins before the characterization harness is real | Silent data corruption in a claims platform serving ~100 customers. Existential to the engagement and to the client relationship. |
| **R3** | Triggers cause invisible side effects during extraction | Same failure mode as R2, arriving through a path EY-P never named. |
| **R4** | Clean-room build fails / repo is incomplete | Discovered in week 1 if we look, month 6 if we do not. Cheap test, enormous information value. |
| **R5** | Droit disengages and system knowledge is lost | The only specification of the system is in contractors' heads. |
| **R6** | Windows Server 2016 (12 Jan 2027) and SQL Server 2017 (12 Oct 2027) both expire mid-programme | Compliance and insurance exposure with ~1m PHI records. EY-P rated this Low. |
| **R7** | Cross-client segregation defect recurs on our watch | Fourth occurrence, now attributable to us as operator. Reputationally worse than any technical slip. |
| **R8** | The remaining 60% automation opportunity is portal/telephony RPA | Where the promised $620k lives, and the hardest class of work to sustain. |
| **R9** | WCF blocks the .NET Core path and requires third-party partner coordination | Schedule risk outside our control, unnamed in the report. |
| **R10** | Granite hires a Director of Engineering who re-litigates our scope | Structural commercial risk of a staff-augmentation-adjacent engagement. Anticipate it; ideally influence the hire. |
| **R11** | Offshore PHI exposure without a masked environment | Blocks our own delivery model until 1.4 lands. |
| **R12** | Reporting coupled to OLTP silently constrains every refactor | Slow, cumulative drag rather than a single event. |

---

## 6. Contradictions in the source report to resolve

The report is a working draft marked "PRELIMINARY DRAFT FOR REVIEW" throughout, and it does not fully reconcile with itself. Each of these changes our scope:

- **Source control:** p.25 says code is on **TFS** with Azure DevOps "evaluated but not initiated." p.44 says Granite "leverages **Azure DevOps** for source control." p.36 lists TFS, Azure Repos/Git, *and* Azure Repos/TFVC as tooling. `[ASSUMPTION]` the Python AI/ML repo is on Azure DevOps while ProWeb remains on TFS — but this must be confirmed, since it determines whether 1.1 is a migration or a consolidation.
- **Inference runtime:** p.31 refers to "**Ollama**-based open source models" while the same page's diagram and p.32 specify **GroqCloud** with Llama 4 Scout and gpt-oss-120b. Ollama is a local runtime; Groq is hosted. Either there is local inference infrastructure nobody described, or this is a drafting error.
- **R&D as % of revenue:** ~9% on pp.7 and 9, ~8% on p.15.
- **Incident count:** management said five, documents showed seven. Already flagged by EY-P, but it sets a baseline for how to weight unverified management statements generally.
- **SQL Server version:** inferred, never confirmed.
- p.26 footnote markers do not match the footnote list. Minor, but consistent with a document whose numbers may still move.

---

## 7. What we need before we price anything

Non-negotiable inputs to a fixed-price conversation:

1. Stored procedure and trigger inventory with complexity metrics (0.2)
2. Successful clean-room build (0.1)
3. Confirmed SQL Server version, edition, licensing (0.4)
4. WCF endpoint inventory with external consumers (0.1 / 0.2)
5. Segregation defect root cause (0.7)
6. Reporting-to-OLTP dependency map (0.9)
7. Droit transition terms (0.10)
8. Close date — everything in the 2027 analysis is relative to it

**Commercial posture:** T&M or capped-T&M for Phase 0 and Phase 1. Fixed price only per domain in Phase 2, and only after 0.2. We should be willing to walk away from a fixed-price commitment on the full extraction — and say so early rather than discovering our position under pressure later.

---

## 8. Open questions

- Is our role remediation delivery, modernization execution, post-close R&D capacity, or a second-opinion read for Park Road? `[ASSUMPTION]` this note assumes delivery takeover.
- Do we displace Droit/Lapiz, sit alongside them, or absorb them?
- What is the expected hold period? It should drive the Phase 2 gate more than any technical factor, and we do not know it.
- Does Park Road expect the ~$620k automation savings in the model? If underwritten, R8 becomes their risk too and we should say so before close, not after.
- Who owns the fourth segregation incident if it happens after we take over? Answer this contractually.
- Is there a BAA with Groq covering PHI in medical records?
- Would Park Road support us influencing the Director of Engineering hire?

---

## Verification note

Microsoft lifecycle dates in §2.3 and R6 were verified against Microsoft Learn on 29 Jul 2026: [SQL Server 2017](https://learn.microsoft.com/en-us/lifecycle/products/sql-server-2017) extended end 2027-10-12; [Windows Server 2016](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2016) extended end 2027-01-12. All Granite-specific figures trace to the EY-P report at the page numbers cited in [[park-road]]. Every inference is tagged `[ASSUMPTION]`; no technical detail about Granite's environment has been supplied where the report is silent.
