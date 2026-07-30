---
type: Note
status: Draft
related_to:
  - "[[park-road]]"
  - "[[park-road-granite-platform-takeover-roadmap]]"
  - "[[park-road-granite-estimate-accuracy-review]]"
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

## The part that surprised us

Add our positions up and the totals broadly agree with theirs. Their six sized items come to roughly 3,880–5,300 hours on the red conversion. Ours land somewhere around 3,100–5,900. The ranges overlap almost entirely.

**What we disagree about is the distribution, not the size.** We would take money out of the .NET upgrade, out of source control, and out of the database data movement, and put considerably more into the frontend while refusing to commit on business logic until someone has opened the database. That is a more useful disagreement than a headline total, and it is a more honest one — we are not claiming EY-P is uniformly wrong; we are claiming they are miscalibrated in both directions: too generous on the mechanical work, and unknowably optimistic on the risky work.

---

## What neither estimate covers

This is where the gap is largest, and it is a scope gap rather than an estimating one.

**There is no automated regression baseline** in EY-P's plan, and there cannot be a credible business logic refactor without one — we put that work at roughly 620 to 1,060 hours on its own. Nothing pays for a **development environment without live patient data**, which is a prerequisite for any offshore development against a platform holding around a million PHI records. **Nothing addresses the cross-client data exposure** that has now recurred three times, in 2022, 2024, and again in January 2026, and whose cause EY-P's own incident description attributes to account numbering collisions — a data model problem, which is why patching it twice did not hold.

Both the data warehouse and the Windows OS upgrade are explicitly left unsized. The second of those has a date on it: Windows Server 2016 leaves extended support on 12 January 2027, roughly five and a half months from now, with SQL Server 2017 following on 12 October 2027. Both expire well before a 22-to-28-month programme completes.

And neither estimate contains any concept of replacing the platform — only of repairing it. That is the difference between the work costing what EY-P says and the investment thesis actually landing.

---

## Proposed phasing

We would structure this as five phases, the first of which is the only one we ==*would price today*==.

**The assessment phase comes first and stands alone.** ==**Six to eight weeks, roughly 600 to 800 development hours (2.5 FTEs 0.5PM, 1 BE, 0.5 DEVOPS)**==, to establish that we can build and deploy the platform ourselves without the incumbent vendor, to map what is actually in the database, to count the screens and the reports, to run Microsoft's migration assessment against a live connection, and to answer the one question that determines everything after it: whether Granite's four solution lines can be separated or whether they share one inseparable core. Most of what makes this engagement risky is unknown rather than difficult, and nearly all of it is cheap to find out. This phase exists to convert unpriced risk into priced risk, and it is where every estimate above either firms up or moves.

**Second, making change safe, over three to four months and roughly 1,600 to 2,400 hours** ==**(3.5 FTEs, 0.5PM, 2 BE, 0.5 DEVOPS)**==**.** The regression baseline, a reproducible build with automated deployment, a development environment without live patient data, the two 2027 support deadlines cleared, the outstanding critical vulnerabilities closed, and application-layer protection in front of an internet-facing platform that currently has none. None of this is modernization. All of it is the precondition for modernization, and it is the part EY-P's plan does not contain.

**Third, separating reporting from the operational database.** Two to three months, 1,300 to 2,600 hours. Power BI queries the transactional database directly today, which means the schema is effectively frozen by report consumers nobody has enumerated. Until that coupling is broken, every subsequent change is negotiating with an unknown set of operational reports. Doing this early is what makes the rest possible, and it is comparatively cheap here because the reporting layer is thin.

**Fourth, replacing the platform one solution line at a time**, over twelve to twenty months. We would start with Discovery — two percent of volume, advisory-only, the client retains billing and collections — deliberately as a pilot, to establish the pattern and the scaffolding rather than to deliver value. Then Aged, at thirty-two percent of volume and the strongest fee rates, as the first line carrying real weight. Then Historical, then Day 1 last, since it owns the full lifecycle and is the largest surface. Throughout, the platform continues to behave identically toward clients, clearinghouses, and third-party data providers. This phase is somewhere between 6,400 and 11,000 hours, and that range is wide enough to be honest rather than precise enough to commit to. We would price it per line, after the assessment, and never as a single number.

**Running alongside all of it, extending the AI capability.** Granite's Python stack — the BioBERT clinical feature extraction, the XGBoost claim scoring, the Groq-hosted document intelligence — is genuinely good and architecturally separate from the legacy platform. It can be extended without touching ProWeb, which makes it the only place we can deliver visible value in the first few months. It is also where the client's return actually lives: EY-P identifies roughly $620k of annual savings available through automation, with only about forty percent of it on the current roadmap. The caution is that a large share of the remaining opportunity is payer portal and telephony work, which is brittle, breaks whenever a payer changes a screen, and carries per-payer maintenance indefinitely. The build is not the cost there. We should re-baseline that opportunity ourselves before anyone underwrites it.

---

## What we would want before committing beyond the assessment

The stored procedure, function, view, and trigger inventory, because it is the only honest basis for the business logic estimate. A successful build from a clean checkout, because a failure turns the first phase into a reconstruction project. A screen count, because the frontend range is twice as wide as it needs to be without one. A report count with named owners, because the reporting work moves by several hundred hours on it. An answer on whether the solution lines separate, because it determines whether we are proposing incremental replacement or a much less attractive in-place extraction. And a live database connection during diligence, because without it the migration assessment and the database decision both slip past close.

Two commercial points to settle internally. We should not quote any single figure before the assessment returns — including our own, and particularly not on business logic. And we should be clear with Park Road that EY-P's dollar figures embed an $85-an-hour contractor rate. The hours are debatable. The price, at anyone else's rates, is not the price.

---

## Verification note

Page 26 was read as a rendered image to distinguish EY-P's printed dollar ranges from the hour ranges annotated in red. All dollar figures, category structure, totals, and footnotes are EY-P's as printed. All hour figures are ours, derived from those dollars; the implied blended rate of $83–88 per hour is our arithmetic on the two sets of numbers, and EY-P's footnote 1 independently states their costs derive from contractor scrum team effort. The 22-to-28-month timeline and the $300–400k leadership hiring figure are EY-P's, from p.11 and p.26 respectively. Microsoft support dates verified against Microsoft Learn on 29 Jul 2026. Our own phase estimates derive from the internal `estimation-reference` and remain provisional pending assessment — six of them scale with counts that do not currently exist in the diligence. The correction in the opening section supersedes the attribution of hour figures to EY-P in [[park-road-granite-platform-takeover-roadmap]], [[park-road-granite-executive-summary-sow]], and [[park-road-granite-estimate-accuracy-review]].
