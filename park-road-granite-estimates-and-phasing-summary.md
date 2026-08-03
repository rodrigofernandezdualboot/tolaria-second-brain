---
type: Note
status: Draft
related_to:
  - "[[park-road]]"
  - "[[park-road-granite-platform-takeover-roadmap]]"
  - "[[park-road-granite-estimate-accuracy-review]]"
_organized: true
---
# Park Road / Granite — Executive Summary: Estimates and Proposed Phasing

## Where our read differs

Taking the six sized items in turn.

**On the .NET framework upgrade we think they are high.** Moving from .NET 4.5 to 4.8.1 and refreshing library dependencies is bounded, well-documented work. Microsoft publishes the breaking changes between framework versions, which makes the compatibility surface knowable rather than exploratory. Their figure implies three to four developer-months; **ours is closer to one to two**. The comparable is a prior migration into MAUI, which was harder and went faster than this scale of estimate would suggest. The caveat is real though: the figure depends on the dependency count and on the deployment workarounds the team has accumulated, neither of which we have seen. This estimate firms up the moment we build the application ourselves.

**On the frontend, we think their range is too narrow, not too high.** This line item is a rewrite of the user interface onto a modern framework, and we have seen exactly one screenshot of the existing application — an appendix page. Their $70–85k implies somewhere between roughly 25 and 50 screens at normal rates. For a platform covering the full claim lifecycle across intake, accounts review, coverage discovery, filing, follow-up, and payment, twice that is entirely plausible. With two front-end developers the honest range is 800 to 2,000 hours, and the top half of that is not pessimism, it is the absence of a screen count.

**On business logic refactoring the number may be fine and the framing is wrong.** Their description scopes this as stored procedures and triggers moving into application and service-layer components. **Two problems. The first is scope**: business logic in a database of this age rarely confines itself to procedures and triggers — functions, views, linked servers, and scheduled jobs commonly hold behavior too, and none of those appear in the line item. **The second is verification**, and it is the more serious one. There is no automated testing anywhere in the application. Moving logic out of the database and claiming feature parity with what it used to do is, without tests, an assertion rather than a demonstrable property. The work could genuinely land inside their 10 to 13 developer-months. It could also be several times that. Until someone has looked at the database structure — the tables, procedures, functions, views, triggers, and linked servers — it is honestly unknowable, and we should say so rather than substituting a number of our own.

**On infrastructure modernization their number is high, but for a subtler reason than it first appears.** Their $60–70k covers upgrading SQL Server or migrating to Azure SQL. Moving the data is the cheap part: taking a backup and restoring it into Azure is roughly a week's work. The catch is that a backup-and-restore requires downtime, and with claims processing running for around a hundred customers that may not be available — which is exactly why the expensive online-migration approaches exist. Nobody has asked Granite whether a maintenance window is acceptable, and the answer swings this line item substantially. A weekend of downtime would take the data movement from a couple of hundred hours to about forty.

What their figure also obscures is the real cost, which is not the migration at all. Azure SQL Database does not support linked servers or SQL Agent jobs, and both very likely exist in a platform that runs its business logic in the database. Cross-database queries and the shift from Windows authentication to Entra ID identities compound it. So the honest breakdown is a small data-movement cost and a much larger compatibility remediation cost — and their single undifferentiated number hides both the cheap path and the expensive dependency. Microsoft's assessment tooling will tell us which target is viable, but it requires a live connection to the database, which makes this an access problem during diligence rather than a desk exercise.

**On source code management, they are well high, and the reason is a scoping question nobody asked.** Migrating from TFS to Azure DevOps sounds like a month or two of work only if the commit history and the work-item data come with it. If Granite simply wants the repositories in Azure DevOps and the team productive on day one, this is a couple of days, not $15–25k. That is roughly a tenfold swing resting on a question neither EY-P nor our own earlier note thought to ask. Worth adding that the report contradicts itself here: page 25 says the code sits on TFS with Azure DevOps merely evaluated, while page 44 says Granite already uses Azure DevOps for source control. If page 44 is right for any part of the estate, some of this is already done.

**On CI/CD we think they are right, and it is worth saying why.** Their $15–25k is realistic for this particular stack. The pipeline they need is not a modern one: a Windows-hosted machine to run the legacy framework build, an artifact uploaded and pulled down onto the target, files replaced into IIS, and service accounts configured with enough permission to do it. That is genuinely a month or two of fiddly work. Had this been a .NET Core application that could be containerized and pushed to a registry, their figure would have been far too high. The cost here is the deployment mechanics a legacy estate imposes, not pipeline configuration — which also means this work gets cheaper if the framework migration happens first, and more expensive if we build the legacy pipeline and then replace it.

---

## Proposed phasing

We would structure this as four phases:

**The assessment phase comes first and stands alone.** ==**Four to six weeks, roughly 600 to 800 development hours (2.5 FTEs 0.5PM, 1 BE, 0.5 DEVOPS)**==, to establish that we can build and deploy the platform ourselves without the incumbent vendor, to map what is actually in the database, to count the screens and the reports, to run Microsoft's migration assessment against a live connection, and to answer the one question that determines everything after it: whether Granite's four solution lines can be separated or whether they share one inseparable core. Most of what makes this engagement risky is unknown rather than difficult, and nearly all of it is cheap to find out. This phase exists to convert unpriced risk into priced risk, and it is where every estimate above either firms up or moves.

**Second, making change safe, over two to three months and roughly 1,200 to 1,680 hours** ==**(3.5 FTEs, 0.5PM, 2 BE, 0.5 DEVOPS)**==**.** The regression baseline, a reproducible build with automated deployment, a development environment without live patient data, the two 2027 support deadlines cleared, the outstanding critical vulnerabilities closed, and application-layer protection in front of an internet-facing platform that currently has none. None of this is modernization. All of it is the precondition for modernization, and it is the part EY-P's plan does not contain.

**Third, separating reporting from the operational database.** ==**(2.5 FTEs, 0.5PM, 2 BE)**== **Two to three months, 800 to 1,200 hours**. Power BI queries the transactional database directly today, which means the schema is effectively frozen by report consumers nobody has enumerated. Until that coupling is broken, every subsequent change is negotiating with an unknown set of operational reports. Doing this early is what makes the rest possible, and it is comparatively cheap here because the reporting layer is thin.

==**Fourth, replacing the platform one solution line at a time**, over ten to twelve months. We would start with Discovery — two percent of volume— deliberately as a pilot, to establish the pattern and the scaffolding rather than to deliver value. Then Aged, at 32% of volume and the strongest fee rates, as the first line carrying real weight. Then Historical, then Day 1 last, since it owns the full lifecycle and is the largest surface. Throughout, the platform continues to behave identically toward clients, clearinghouses, and third-party data providers. This phase is somewhere between **8,000 and 11,000 hours**, and that range is wide enough to be honest rather than precise enough to commit to. We would price it per line, after the assessment, and never as a single number. **(5 FTEs, 0.5PM, 2 BE, 1FE, 1QA, 0.5 DEVOPS)**==

**This is also where EY-P's frontend modernization lives.** We do not run a separate user-interface rewrite, because each solution line gets its new interface as part of its own replacement. Rewriting screens onto a modern framework and then replacing those same screens during displacement means paying twice for the same work — which is why the frontend appears here as a component of four deliverables rather than as a project of its own. The dedicated front-end capacity in this phase is what carries it.

That does mean the 800 to 2,000 hours we put on the frontend is folded inside the 8,000 to 11,000 above, as part of treating each displaced line as a new build. It needs to be **visibly allocated per line rather than left implicit**, for two reasons. It is too large a number to sit unstated, and anyone comparing our plan to EY-P's will notice that frontend modernization has no line of its own and ask where it went. Better to answer that in the plan than in the meeting.

**Running alongside all of it, extending the AI capability.** Granite's Python stack — the BioBERT clinical feature extraction, the XGBoost claim scoring, the Groq-hosted document intelligence — is genuinely good and architecturally separate from the legacy platform. It can be extended without touching ProWeb, which makes it the only place we can deliver visible value in the first few months.

---

## Where each of EY-P's investment areas lands

The eight investment areas from page 26, mapped onto the phases above.  

| EY-P investment area (p.26) | EY-P est. | Where it lands |
| --- | --- | --- |
| .NET framework modernization | $35–60k | **Phase 2**  |
| Frontend architecture & UI modernization | $70–85k | **Phase 4**  |
| Business logic refactoring | $140–190k | **Phase 4** prerequisites in **Phase 1**  and **Phase 2**  |
| Infrastructure modernization | $60–70k | **Phase 1** assess, **Phase 2** in-place upgrade |
| Source code management | $15–25k | **Phase 2**  |
| CI/CD pipeline | $15–25k | **Phase 2**; modern pipeline for new services in **Phase 4** |
| Data warehouse | *not sized* | **Phase 3**  |
| Windows OS upgrade | *not sized* | **Phase 2**  |

Two things the mapping makes visible.

**Business logic and frontend land on the identical four deliverables.** EY-P treats them as two separate projects totalling $210–275k; we treat them as one activity performed four times, once per solution line.

**Five things in our plan have no page 26 equivalent at all:** the assessment phase itself, the regression baseline, the masked development environment, the layer that keeps ProWeb's external behaviour unchanged during replacement, and the retirement of displaced components.

---

## What we would want before committing beyond the assessment

- The stored procedure, function, view, and trigger inventory, because it is the only honest basis for the business logic estimate. 
- A successful build from a clean checkout, because a failure turns the first phase into a reconstruction project. 
- A screen count, because the frontend range is twice as wide as it needs to be without one. 
- A report count with named owners, because the reporting work moves by several hundred hours on it. 
- An answer on whether the solution lines separate, because it determines whether we are proposing incremental replacement or a much less attractive in-place extraction. 
- A live database connection during diligence, because without it the migration assessment and the database decision both slip past close.

---

## Total estimated cost
| \# | Hours |  |
| --- | --- | --- |
| Phase 1 | 600-800 | Discovery |
| Phase 2  | 1200-1680 | Modernization |
| Phase 3 | 800-1200 | Reporting preparation (Added Scope) |
| Phase 4 | 8000-11000 | Legacy migration (Future development) |
| Total (1&2) | 1800-2480 |  |
