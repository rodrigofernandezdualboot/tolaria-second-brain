# Roofr

> **Status:** pre-discovery, updated 18 Aug 2026. Stack assumption **settled: LAMP / MySQL**.
> Working documents live in the Sol project folder:
> `engagements/roofr-heroku-exit/zero-downtime-options.md` (options + risks) and
> `engagements/roofr-heroku-exit/poc/` (working proof of concept + run report).

## The deal

AWS-originated, Blaine Wagner's first. Roofr is a **roofing CRM** — B2B SaaS, PHP/Laravel
monolith with a separated SPA front end and one or two small services (Node for PDF rendering).
~100 engineers on the codebase, ~70 active day to day.

Two drivers came out of the 11 Aug intro call. Only one has a date attached.

1. **Heroku exit.** Off Heroku by Christmas, targeting early December. Zero-downtime database
   migration is the core concern, named by Kevin Prince and John (VP Engineering / CTO).
   Dump-and-load is off the table.
2. **AI spend governance.** ~70% of AI spend is engineering with no attribution by team or
   workstream. Non-engineers default to frontier models with no guardrails. The Anthropic
   relationship is a frustration at their scale — high spend thresholds, weak enterprise
   controls. They plan to route through AWS (Bedrock and Amazon Q) for attribution, cost
   visibility and security. Separately, they lose developer-experience tracking data when PRs
   merge to main. Product AI is a maybe, timing uncertain.

**Hard constraint:** no PHP or Laravel modernization. Kevin was explicit — modern Laravel shop
(DTOs, clean patterns), prefers "usual LAMP-stack scaling problems" over microservice sprawl.

## Timeline — the December date is deliberate

| Date | What it is |
|---|---|
| Jun 1 – Nov 30 | Atlantic hurricane season. Storm damage drives their demand. **This is their peak.** |
| Early December | Target cutover — the first quiet window *after* peak. |
| Christmas | Kevin's stated hard deadline. |
| Mid-January | Heroku contract expiry. The commercial backstop. |

Do not read December as "before the peak" — it is after it. Two consequences: rehearsals fall in
Sept–Nov when the platform is busiest and their engineers least available, and the mid-January
contract plus the CFO's approved dual-infrastructure spend gives roughly six weeks of **paid
rollback runway** after cutover. That runway is the answer to their biggest fear.

## Stack

**Application** — PHP/Laravel monolith, modern patterns (DTOs). Separated SPA front end.
One or two small services, e.g. Node for PDF rendering. Deliberately avoiding microservice
sprawl. Documentation exists but is messy; spec volume is a growing internal concern.

**Database — MySQL (LAMP).** Settled assumption as of 18 Aug 2026, consistent with Kevin's own
"usual LAMP-stack scaling problems" framing. **Heroku has no first-party MySQL**, so their
database is either a third-party add-on (JawsDB or ClearDB) or MySQL hosted outside Heroku.
That distinction is now the top open question — see below.

**Hosting** — Heroku today, AWS as the target, with AWS's own team involved in the deal.

**Spatial data** — they store GeoJSON in the database and flagged it as *large*. On MySQL that
means either a `JSON`/`LONGTEXT` column or native spatial (`GEOMETRY`/`POLYGON`). Which one it
is changes the target schema and the verification strategy, and it is still unanswered.

**AI** — Anthropic direct today (~70% of the engineering AI budget), moving toward Bedrock and
Amazon Q for attribution, guardrails and cost visibility.

## The core technical finding

Roofr told us there is **no replication support on Heroku**, and that holds for the MySQL path
independently: AWS DMS MySQL change data capture requires `binlog_format=ROW`,
`binlog_row_image=FULL`, retained binary logs, and `REPLICATION SLAVE` + `REPLICATION CLIENT`
grants. JawsDB documents a restricted-privilege service with SUPER limitations and no binlog
access; assume ClearDB grants nothing. Those grants are server-scoped, which is precisely why
managed add-on providers withhold them.

So the plan is **not** log-based CDC. It is:

1. **Backfill** the data that already exists — bounded, resumable, throttled, and crucially
   never overwriting rows the change stream has already delivered.
2. **Trigger-based change capture** for everything written during the transition — `AFTER`
   triggers on the source writing keys to a change-log table, drained to RDS by a shipper.
   Needs only `TRIGGER` plus ordinary DML on their own schema, which any managed provider grants.
3. **AWS DMS still has a job** — full load needs no CDC privileges, so use DMS for the initial
   bulk copy and its Data Validation for evidence. Keeps AWS's motion intact on their own deal.

Proven end to end in the POC: 401 ms application write pause, zero divergence across tens of
thousands of rows and thousands of geometry values.

## Open questions, in priority order

1. **Who hosts your MySQL today — JawsDB, ClearDB, or somewhere else? What plan?**
   If it is JawsDB, it already runs on AWS infrastructure. That could materially shorten the
   database half of this project and is worth establishing before anyone estimates it.
2. **Is the GeoJSON in a `JSON`/`LONGTEXT` column or a native MySQL spatial column?**
   Answerable by their new AI lead in one message. Changes the target schema and verification.
3. **How large is the database, and what is the peak write rate?** Backfill wall-clock is a
   direct function of these, and it is the single biggest input to the migration schedule.
4. Exactly what was asked of Heroku / the add-on provider about replication, by whom, and when?
5. Common Runtime or Private Space? Which region? Full add-on inventory.
6. Who allowlists your outbound IPs, and who posts webhooks to you?
7. Any table without a primary key? Any generated columns or spatial indexes?
8. Can schema migrations be gated for a short window, across ~70 daily contributors?

## Risk nobody has priced yet

~70 engineers merging daily against a replication window that cannot absorb arbitrary DDL.
A blanket schema freeze across a 100-engineer org during their busiest quarter is not
enforceable. The answer is a migration-review gate plus an automated drift guard, not a freeze —
and raising it early is the fastest route to credibility with John.

## People

- **Kevin Prince** (kprince@roofr.com) — driving it; set the no-modernization line.
- **John** — Roofr VP Engineering / CTO; co-owner of the zero-downtime concern.
- **New internal AI lead** — founding engineer just moved into the role, deep codebase context.
  Natural champion for both workstreams and the fastest path to technical answers.
- **AWS** — hcsinger@amazon.com, sbkhadka@amazon.com ("Hart's team").
- **Dualboot** — Billy Boozer and Rodrigo on the next call; 30-minute internal prep beforehand.

## Prior-client example Kevin asked for

Dataplor Heroku migration — SOW plus "Assess Findings and IW Migrate Plan", posted by Mike
Dennis in #roofr-dealteam-sales (Slack files `F0BQCKU3NC8`, `F0BPJ8ZKSRY`; not in the Drive
folder). Mission ran that assessment; Mike flagged TD Synnex as our route "if we are in play for
this part". **Open commercial question: who performs the migration assessment.** Settle it
internally before the client call.

## Commercial note

The HubSpot deal of record is "Roofr <> AI Platform (Bedrock & Anthropic)" — the AI framing —
while the urgent, dated, technically risky work is the Heroku exit. Two SOWs, or the December
deadline swallows the AI workstream.
