---
type: Note
status: Draft
related_to:
  - "[[park-road]]"
  - "[[park-road-granite-platform-takeover-roadmap]]"
  - "[[park-road-granite-executive-summary-sow]]"
_width: wide
_organized: true
---
# Park Road / Granite — Estimate Accuracy Review (EY-P p.26)

**Audience:** Dualboot internal.
**Purpose:** reconcile three views of EY-P's tech debt remediation estimates — EY-P's own figures from p.26, the derivation in [[park-road-granite-platform-takeover-roadmap]], and Rodrigo's practitioner read. Where they disagree, name what resolves it.
**Status:** Rodrigo's assessment is captured as given. My reconciliation and the corrections in §4 are mine and are open to challenge.

Two transcription readings I've assumed: **"DFS" = TFS** (Team Foundation Server), and **"Azure shops or SQL shops" = SQL Agent jobs**. Correct me if either is wrong — the second one carries real weight in §3.4.

---

## 1. Reading the numbers consistently

EY-P states hours on p.26; Rodrigo's read is in months. At ~170 productive hours per developer-month the two reconcile, and **his conversions check out against EY-P's published hours**, which matters because it means the disagreements below are about effort, not arithmetic.

| EY-P item | EY-P hours | Implied dev-months | Rodrigo's reading |
| --- | --- | --- | --- |
| .NET framework modernization | 400–700 | 2.4–4.1 | "three and four months" ✓ |
| Frontend architecture & UI | 820–1,000 | 4.8–5.9 | "five to six months" ✓ |
| Business logic refactoring | 1,600–2,200 | 9.4–12.9 | "10 and 14 months for a single developer" ✓ |
| Infrastructure modernization | 700–800 | 4.1–4.7 | "four and five months" ✓ |
| Source code management | 180–300 | 1.1–1.8 | "one and two months" ✓ |
| CI/CD pipeline | 180–300 | 1.1–1.8 | "one to two months" ✓ |

One normalization caveat: EY-P's figures are single-developer effort, and Rodrigo's frontend view assumes **two** front-end developers. Everything below is stated as hours to keep capacity assumptions out of the comparison.

---

## 2. The three views side by side

| Item | EY-P | Roadmap note (mine) | Rodrigo | Direction of disagreement |
| --- | --- | --- | --- | --- |
| .NET 4.5 → 4.8.1 + dependencies | 400–700h | 200–400h | **170–340h** | Both below EY-P. We converge. |
| Frontend / UI modernization | 820–1,000h | 820–1,000h *(adopted EY-P)* | **800–2,000h** | Rodrigo widens the top end sharply. **I was too accepting.** |
| Business logic refactoring | 1,600–2,200h | 2,400–4,800h | **Unknowable; may be fine** | Rodrigo is more measured on the number, harder on the risk. **My figure was overconfident.** |
| Infrastructure / database | 700–800h | 300–600h in place, +400–800h cloud | **"Way too high"; restore ≈ 1 week** | Agreement once data movement is separated from remediation. |
| Source code management | 180–300h | 120–240h | **≈16–40h if repos only** | Depends on a scoping decision neither of us stated. |
| CI/CD pipeline | 180–300h | 160–320h | **EY-P is accurate** | We converge at EY-P's figure. |

---

## 3. Item by item

### 3.1 .NET framework modernization — we both say EY-P is high

**Rodrigo:** 400–700h is too high for a 4.5 → 4.8.1 move plus dependency updates. One to two months, conditional on seeing the code, the dependency count, and the deployment workarounds. Comparable: a prior migration into MAUI, which was harder. Framework changelogs plus AI assistance make this fast.

**Reconciliation:** my 200–400h and his 170–340h overlap substantially. Both sit below EY-P. Call the working range **200–400h**, narrowing to his figure if 0.1 shows a clean dependency graph.

**The one thing to be careful about.** His speed argument rests partly on AI assistance. Our `estimation-reference` gates AI velocity credit on `stackFamiliarity = Confirmed` and caps complex-logic gains at −5 to −10%; a framework version bump reads closer to the "architecture & DevOps setup" tier at −10 to −15%. Getting from EY-P's 400–700h to 170–340h is roughly a 50–60% reduction, which is well beyond what the reference permits — and beyond its own anti-overoptimism ceiling of 30% per epic.

That does not make him wrong. It means **the claim needs the evidence attached.** The reference allows higher rates only where prior-project evidence demonstrates them, and the MAUI migration is exactly that evidence — it is just not written down anywhere. See §6.

**Resolved by:** 0.1 (build validation, dependency graph) and capturing the MAUI comparable.

### 3.2 Frontend modernization — Rodrigo is right and my note was wrong

**Rodrigo:** this is a rewrite to a modern framework, and we have seen **one screenshot** (p.48, appendix). No feature count, no screen count. With two front-end developers the honest range is 800–2,000h. "It's okay but we need more information."

**Where I was wrong.** My roadmap note adopted EY-P's 820–1,000h directly and listed frontend as a point of agreement with EY-P. That was unjustified. EY-P's figure implies roughly 25–50 screens at our reference's rates (8–16h simple, 20–40h complex); 2,000h implies 50–100. For a full-lifecycle RCM platform with intake, review, coverage discovery, filing, follow-up, and payment workflows, the higher number is entirely plausible. **Nobody has counted.**

Our own estimation reference is explicit here: never skip screen-count calibration when a screen inventory is available — and never substitute a gut estimate when actual counts are known. We have no inventory, so the range must stay wide rather than borrow EY-P's precision.

**Correction to make:** frontend range becomes **800–2,000h**, flagged count-driven, and §2.3 of the exec summary can no longer cite frontend as agreement with EY-P.

**Resolved by:** a screen and feature inventory — **which is missing from the Phase 0 task list.** See §5.

### 3.3 Business logic refactoring — Rodrigo's framing is better than mine

**Rodrigo:** EY-P scopes this as stored procedures moving to the application layer. The actual surface is likely wider — functions, views, triggers, linked servers. Because there is no testing anywhere in the application, we cannot guarantee the relocated logic achieves feature parity with what the database was doing. "It can be very dark." Their 10–14 months "could be okay again depending on what we have in the database."

**Where I overreached.** My note asserted 2,400–4,800h, roughly 1.5–2× EY-P, derived from 18 workflow steps at complex-logic rates with the porting rule at 90–100% of new build. The derivation is defensible, but presenting it as a range implied a precision the evidence does not support. Rodrigo's position is the more honest one: **the number is unknowable until we see the database, and the binding constraint is verification rather than volume.**

**This strengthens the more important argument.** Our case against EY-P was never really that their hours are too low — it is that they scoped the work without scoping the means to do it safely. Rodrigo's point sharpens that from two directions:

- **Scope omission.** EY-P's line item says stored procedures. Functions, views, triggers, and linked servers are not named on p.26 or p.25. If logic lives in those too, their scope is incomplete before their estimate is even considered.
- **No verification path.** Without tests, "feature parity" is an assertion, not a demonstrable property. This is the regression-baseline argument, and it is the one to lead with.

**Correction to make:** replace the 2,400–4,800h assertion with a stated range plus an explicit unknowability flag, and move the emphasis from hours to scope and verification.

**Resolved by:** 0.2, extended to name functions and views explicitly alongside procedures and triggers.

### 3.4 Infrastructure modernization — not a real disagreement once unbundled

**Rodrigo:** 700–800h is way too high. The simplest path is a backup restored into Azure — perhaps a week. But backup/restore requires downtime, so it may not be available; worth asking whether Granite would accept a window. Separately: **linked servers and SQL Agent jobs are not supported on Azure SQL Database** and require changes to work around. Microsoft's assessment tool can tell us whether migration is viable, **but it needs a live database connection.**

**Reconciliation.** He and I are describing different halves of the same task and reaching the same conclusion. Moving the data is cheap. The cost is compatibility remediation — precisely the SQL Agent jobs and linked servers he names, plus cross-database queries and the Windows-to-Entra identity change. My +400–800h was almost entirely remediation, not data movement. Stated properly:

| Component | Estimate |
| --- | --- |
| Data movement — backup/restore, with downtime | ~40h |
| Data movement — online, no downtime (DMS or replication) | 120–240h |
| Compatibility remediation (SQL Agent jobs, linked servers, cross-database queries, identity) | 300–700h, count-driven |
| In-place SQL Server 2022 upgrade instead of cloud | 300–600h |

So EY-P's 700–800h is not absurd as a *total* — it is roughly our remediation plus online migration. What is wrong is that it is presented as one undifferentiated number, hiding both the cheap path and the expensive dependency.

**Two things Rodrigo adds that I did not have.**

The **downtime question is commercially live**. If Granite will accept a maintenance window, the migration path gets dramatically cheaper. Nobody has asked. Given claims processing for ~100 customers this may be a hard no, but a weekend window is not obviously impossible and it is worth a direct question.

The **assessment tool needs a live connection**, which makes it an access dependency rather than a desk exercise. Task 0.11 currently reads as though we can assess from documentation. We cannot. That has pre-close implications: either we get a live connection or a restored copy during diligence, or 0.11 slips to post-close and the database target decision slips with it.

**Resolved by:** 0.11 with live database access, plus the downtime question to Granite.

### 3.5 Source code management — the estimate hinges on a scoping decision

**Rodrigo:** if history and work-item data do not need to carry over from TFS, and they simply want the repositories in Azure DevOps ready to work on day zero, this is a couple of days. EY-P's 1–2 months is way high.

**Reconciliation.** My 120–240h silently assumed history migration; his ~16–40h assumes repos only. Neither of us stated the assumption, which is the actual finding. This is not an estimating disagreement at all — it is an unasked scoping question with roughly a 10× swing.

| Scope | Estimate |
| --- | --- |
| Repositories only, no history, no work items | 16–40h |
| Plus commit history migration | 80–160h |
| Plus work items and build definitions | 160–300h |

**Worth noting:** p.44 of the EY-P report says Granite already uses Azure DevOps for source control, while p.25 says TFS with Azure DevOps "evaluated but not initiated." If p.44 is right for any part of the estate, some of this work may already be done. That contradiction is already logged in the roadmap note and it directly affects this line item.

**Resolved by:** confirming current-state source control, and asking whether history matters.

### 3.6 CI/CD — Rodrigo endorses EY-P, and explains why

**Rodrigo:** 1–2 months is realistic *for this stack*. Likely shape: a virtual machine to run the old framework build, artifact upload, download onto the target machine, file replacement into IIS, and permission configuration for the account performing it. For a modern .NET Core application that could be containerized and deployed to a registry and container apps, EY-P's figure would be far too high.

**Reconciliation.** My 160–320h sits just above EY-P's 180–300h; he lands on EY-P's figure. Take **180–300h**. He has the better reasoning: the cost is not pipeline configuration, it is the manual deployment mechanics that a legacy IIS estate forces.

**The sequencing consequence.** CI/CD cost is a function of the framework decision. If the platform eventually moves to .NET Core with containerized deployment, this work gets substantially cheaper — but building the pipeline first and migrating the framework second means paying for the legacy pipeline and then replacing it. Worth deciding deliberately rather than by default.

---

## 4. What changes in our position

Three corrections, all of them mine.

**Frontend is no longer a point of agreement with EY-P.** The range widens to 800–2,000h and becomes count-driven. The exec summary's §2.3 currently cites frontend as a place we adopt their number; that needs removing.

**The business logic figure comes down in confidence, not in value.** 2,400–4,800h stays as a reasoned range but gains an explicit unknowability flag, and the argument moves from hours to scope omission and the absence of a verification path.

**The headline framing needs adjusting.** My exec summary leans on EY-P being undersized. Rodrigo's read shows they are **miscalibrated in both directions** — too high on the mechanical work (.NET, source control, the database data movement), and unknowable on the risky work (business logic, frontend). That is a more defensible and more interesting position than "their numbers are too low," and it is harder to dismiss as a vendor talking its own book.

What survives unchanged, and is now the load-bearing argument: **their scope is incomplete.** No regression baseline, no masked development environment, no triggers or functions in the business logic line item, no displacement concept. And the capacity arithmetic still holds — ~3,880–5,300h across a stated 22–28 months is roughly one full-time engineer, whatever the individual line items turn out to be.

---

## 5. Phase 0 changes this generates

| \# | Addition | Why | Estimate affected |
| --- | --- | --- | --- |
| **new** | **Screen and feature inventory** — count and classify ProWeb screens by complexity | The reference's screen-count calibration cannot run without it, and we have one screenshot. Currently absent from Phase 0 | Frontend: 800–2,000h → narrows |
| 0.2 | Extend explicitly to **functions and views**, not just procedures and triggers | EY-P's line item names procedures only; Rodrigo expects the surface to be wider | Business logic |
| 0.11 | Add **live database connection** as a hard prerequisite | Microsoft's assessment tool requires it. Currently written as though desk-assessable | Database target — and its timing |
| 0.1 | Add **dependency graph capture** as a named output | Rodrigo's .NET figure is conditional on dependency count and deployment workarounds | .NET modernization |
| **new** | Confirm **current-state source control** and whether history/work items must carry over | ~10× swing, and p.25 and p.44 of the report contradict each other | Source code management |

---

## 6. The AI velocity question — worth resolving properly

Rodrigo's .NET figure implies a larger AI-assisted gain than our `estimation-reference` currently permits: roughly 50–60% against a reference ceiling of 30% per epic, and against rates that are set to 0% entirely when stack familiarity is unconfirmed.

The reference does provide a route to claiming more, and it is the right one: higher rates apply where **prior-project evidence demonstrates them**, confirmed rather than assumed. Rodrigo has that evidence in the MAUI migration. It is not recorded anywhere, so we cannot cite it.

Two actions, both cheap and both useful beyond this deal:

**Write up the MAUI migration as an internal comparable** — scope, framework versions, dependency count, actual hours, and what the AI assistance concretely replaced. That converts a practitioner's judgment into a citable calibration point.

**Then decide whether the reference's rates need updating**, rather than overriding them case by case. If framework-version migration with published changelogs is systematically faster than the reference assumes, that is a reference bug worth fixing once — not an argument to have on every deal.

Until then: hold the working estimate at 200–400h and record Rodrigo's 170–340h as the optimistic case with the MAUI comparable named as its basis. This also closes the open question in the exec summary about whether we have internal comparables at all. For the T-SQL-to-service-layer extraction, we still do not.

---

## 7. Questions for Granite

Generated by this review, in rough order of how much they move a number.

1. **Would you accept a maintenance window for the database migration?** A weekend of downtime replaces an online migration with a backup and restore. Understood that ~100 customers makes this hard, but nobody has asked.
2. **Can we have a live database connection, or a restored copy, during diligence?** Without it the migration assessment slips past close, and the database target decision slips with it.
3. **Where does source control actually live today** — TFS, Azure DevOps, or both? The report says both.
4. **Does TFS history and work-item data need to carry forward?**
5. **How many screens does ProWeb have, and which are data-heavy?** A rough count from the team is enough to start.
6. **Beyond stored procedures and triggers, what else holds business logic** — functions, views, linked servers, scheduled jobs?

---

## Verification note

EY-P hours in §1 and §2 are from p.26 of the EY-P Project Granite TDD, 12 May 2026. Month conversions use ~170 productive hours per developer-month and reconcile with Rodrigo's stated ranges. Rodrigo's assessments in §3 are captured as provided, with two transcription readings assumed and flagged at the head of this note. Reconciliations, corrections in §4, and the derived component breakdowns in §3.4 and §3.5 are mine. The AI velocity constraints in §3.1 and §6 are quoted from the internal `estimation-reference` — the 30% per-epic ceiling, the 0% rate for unconfirmed stack familiarity, and the prior-project-evidence requirement for higher rates. Azure SQL Database feature limitations referenced in §3.4 (SQL Agent jobs, linked servers) are consistent with Microsoft's migration overview, already cited in the roadmap note. All estimates remain provisional pending Phase 0.
