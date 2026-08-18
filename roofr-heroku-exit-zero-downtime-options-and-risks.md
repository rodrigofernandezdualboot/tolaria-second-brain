---
type: Note
belongs_to: "[[roofr]]"
related_to: "[[roofr]]"
_width: wide
---

# Roofr — Heroku Exit: Zero-Downtime Options and Risks

Prepared for the follow-up technical call (Roofr / Dualboot / AWS).
**Revision 3, 18 Aug 2026** — the stack question is settled: Roofr run a **LAMP stack, so MySQL**. Every Postgres/MySQL fork in revision 2 has collapsed to the MySQL case, and the constraint argument is now sharper rather than weaker. Revision 2 corrected revision 1's seasonality error; see §2.

Sources: `roofr.md` (Tolaria), #roofr-dealteam-sales (Slack), "Roofr Meeting Notes 8/11" (Roofr - Sales Drive folder), plus vendor documentation. Claims marked **[open]** are still unverified with Roofr.

---

## 1. What Roofr actually asked for

Kevin Prince raised two drivers on the 11 Aug intro call. Only one has a date attached.

**Driver 1 — Heroku exit.** Off Heroku by Christmas, targeting early December. Zero-downtime database migration is the core concern, named specifically by Kevin and John (VP Engineering / CTO). Roofr has already told us **there is no replication support on Heroku**, and that **large GeoJSON in database tables may complicate the move**. Dump-and-load is off the table. Kevin asked for a concrete methodology and prior client examples, explicitly not hand-wavy reassurances.

**Driver 2 — AI spend governance.** ~70% of AI spend is engineering with no attribution by team or workstream. Non-engineers default to frontier models with no guardrails. The Anthropic relationship is a frustration at their scale — high spend thresholds, weak enterprise controls. They already plan to route through AWS (Bedrock and Amazon Q) for attribution, cost visibility, and security. Separately, they are losing developer-experience tracking data when PRs merge to main. Product AI is a maybe, with uncertain timing.

**Steer around:** PHP modernization. Kevin was explicit — strong modern Laravel shop with DTOs and clean patterns, no appetite to replatform, happy to take "usual LAMP-stack scaling problems" over microservice sprawl.

### The ask underneath the ask

Roofr already knows the hard part. They have diagnosed their own blocker correctly — Heroku will not give them a change stream — and they are shopping for someone who can still deliver zero downtime with that constraint in place. Kevin's "not hand-wavy" instruction is the actual brief: **the deliverable for the next call is a named mechanism, not a phased-approach diagram.**

That is a favourable position. It means the winning move is not to run discovery at them. It is to arrive with the mechanism that works when replication is unavailable, the evidence that we understand why it is unavailable, and the two questions whose answers change the design.

---

## 2. Correction: the December date is well chosen, not reckless

Revision 1 of this document argued for pulling the cutover earlier because it read the Tolaria summary as "December, then peak." That is wrong, and it would have been an embarrassing thing to say in the room.

The real shape:

| Date | What it is |
|---|---|
| June 1 – Nov 30 | Atlantic hurricane season. Storm damage drives their customer demand. This is their peak. |
| Early December | Target cutover. Deliberately chosen for the first quiet window after peak. |
| Christmas | Kevin's stated hard deadline — must be off by then. |
| Mid-January | Heroku contract expiry. The commercial backstop. |

Two consequences that are better than the revision-1 framing:

**They have ~6 weeks of paid rollback runway.** The Heroku contract runs to mid-January and the CFO has already agreed to absorb a short-term dual-infrastructure spike. Keeping Heroku fully provisioned and warm as a rollback target through early January costs nothing that has not already been approved. Say this out loud on the call — it converts their biggest fear into a bounded one.

**Rehearsals have to happen during peak season.** September through November is when the platform is busiest and their engineers are least available. The cutover window is calm; the preparation window is not. That is the real schedule risk, and it is a staffing problem more than a technical one.

**[open]** Storm-damage roofing demand lags the storm — claims, inspections, and repairs run for months after a landfall. A major late-October hurricane could leave December busier than the seasonal average suggests. Worth asking what their December looked like in each of the last three years rather than assuming the calendar.

---

## 3. Zero downtime is five cutovers, not one

Their architecture is a Laravel monolith with a separated SPA front end, plus one or two small services (Node for PDF rendering). So the switches are:

1. **Data** — the moment writes stop going to Heroku and start going to AWS. The only genuinely hard one, and the one they named.
2. **Monolith web tier** — dynos to ECS/EKS/EC2 behind an ALB. Weighted DNS or a proxy fronting both. Low risk.
3. **SPA front end** — likely already static-hosted, but its API base URL and CORS/CSP origins have to flip in step with the API. A stale cached bundle pointing at the old origin is a self-inflicted outage.
4. **Background work** — queue workers, Heroku Scheduler, the PDF-rendering service. The risk here is double-execution or a silent gap during dual-run, not downtime.
5. **Edges** — outbound static IPs on partner allowlists, inbound webhooks pointed at Heroku hostnames, OAuth redirect URIs, TLS certs, DNS TTLs. These break *after* an apparently successful cutover, which is the worst possible time to find them.

**Define the service level explicitly, in the SOW.** Three honest options:

- **True zero downtime** — no failed requests, no read-only period. Needs forward change replication plus a reversible flip.
- **Zero data loss, brief write-pause** — reads served throughout, writes paused or queued for a bounded window measured in seconds to a few minutes.
- **Announced maintenance window** — 10 to 30 minutes, which is what a dump-and-restore of a database of any real size actually costs. Roofr has ruled this out in substance by ruling out dump-and-load.

Recommendation: **contract the second, engineer toward the first, and price the gap as an explicit option.** Kevin will respect a precise commitment more than an ambitious one — his complaint about the market is exactly that people oversell this.

---

## 4. Why no replication is available to them — the evidence

Roofr stated this. We verified it independently, which is worth demonstrating because it establishes that we understand the constraint rather than just repeating their words.

They run LAMP, so the database is MySQL — and the first consequence is that **Heroku has no first-party MySQL.** Their database is a third-party add-on (JawsDB, ClearDB) or MySQL hosted outside Heroku. Establish which before anything else: JawsDB already runs on AWS infrastructure, and if their instance is already RDS-backed, the database half of this project could be materially shorter than anyone is currently assuming.

**Why DMS change data capture is unavailable either way:**

- DMS MySQL CDC requires `binlog_format = ROW`, `binlog_row_image = FULL`, retained binary logs, and `REPLICATION SLAVE` plus `REPLICATION CLIENT` grants.
- JawsDB documents a restricted-privilege managed service, notes SUPER-permission errors on restore, and does not document binlog access or external replication. Shared plans additionally cap hourly query volume, which a full-load read would breach. Single-tenant plans are the only plausible candidates.
- If it is ClearDB, assume no binlog access at all.
- Those grants are **server-scoped**. That is precisely why a managed provider will not hand them over — it is a product boundary, not an oversight anyone can escalate around.

**The asymmetry that makes Option A work:** the trigger mechanism needs `TRIGGER` on their own schema plus ordinary DML. `TRIGGER` is *schema*-scoped, and every managed provider grants it. Server-scoped privileges are withheld; schema-scoped privileges are granted. That single distinction is the technical heart of the recommendation, and it is the thing to say out loud in the room.

**Given their view that "Salesforce's public stance on Heroku doesn't match what's happening behind the scenes," do not build the plan on their provider's cooperation.** They may already have asked and been refused. Confirm what was asked and what the answer was, then design as if the answer is no. Treat any privilege grant as upside.

### The spatial-data constraint layered on top

Roofr said "large GeoJSON in DB tables." Two very different problems hide behind that phrase:

- **A native MySQL spatial column** (`GEOMETRY`, `POINT`, `POLYGON`) — AWS DMS maps MySQL spatial types **to `BLOB`** on a heterogeneous target. Silent corruption, not an error. This is a strong argument for keeping the migration homogeneous MySQL-to-MySQL, and for verifying spatial values as bytes rather than as re-parsed text.
- **A `JSON` or `LONGTEXT` column** — no type-mapping risk at all. The problem reduces to volume: large row payloads, LOB handling, and throughput.

One more thing worth knowing before anyone writes target DDL: MySQL applies SRID 4326's declared (lat, long) axis order where PostGIS uses (long, lat). It only bites on a cross-engine move, but it bites silently.

**[open] Which is it?** Their new AI lead — a founding engineer with deep codebase context — can answer this in one message. It is the highest-value question on the list and the fastest to close.

### Other DMS constraints that will bite this specific workload

- Every captured table needs a primary key, or DMS silently drops UPDATE and DELETE on it. Laravel pivot tables are the usual offenders.
- **`AUTO_INCREMENT` values are not migrated** by any tool. A counter left at 1 produces duplicate-key errors minutes after cutover under production write load. MySQL does advance `AUTO_INCREMENT` when you insert an explicit higher id, so it partly self-corrects — do not rely on that; set it explicitly and verify.
- **Native `ENUM` columns are not migrated** by DMS at all. A Laravel schema using `enum` needs a look.
- **Changes to `GENERATED` columns are not captured** during CDC. Check whether their schema uses any.
- Indexes on partial columns are not migrated; partitioned tables are created as plain tables on the target.
- Binary logs over 4 GB cause failures, and large transactions need breaking up. "Large GeoJSON" makes big transactions likely.
- **Several DDL forms are not captured during CDC, and some suspend the affected table's replication entirely.** See R2 — with ~70 engineers in the codebase daily, this is the constraint that shapes the whole design.

---

## 5. Options

Reordered from revision 1 now that "no replication" is a stated fact rather than a risk.

### Option A — Trigger-based change capture on the source **(recommended primary)**

`AFTER INSERT/UPDATE/DELETE` triggers on the tables that matter write row keys into a change-log table. A shipper drains that log and applies changes to RDS. Runs with ordinary table-owner privileges — no superuser, no binlog, no replication slot, nothing Heroku has to agree to.

- **Why this is the answer for the call.** It is a named, concrete mechanism that works under the exact constraint Roofr has already identified. It is the thing Kevin asked for.
- **It respects the no-modernization line.** The triggers and the change-log table are additive schema objects with a documented teardown. No business logic changes, no framework changes, no PHP touched.
- **Costs and design work:** write amplification on the source during the window — measure against their peak write rate before committing; ordering and idempotency need real design; tables without a stable unique key need separate handling; the shipper has to be built, monitored, and proven correct. For large GeoJSON rows, ship keys and re-read the payload rather than copying it into the log twice.
- **Verdict:** primary plan. Present it as the mechanism, with the measurement it depends on named.

### Option B — Bounded write-pause at the flip **(the floor under Option A)**

With A running underneath, the final step is: pause writes, drain the change log and the queues, verify, repoint, resume. The pause is the drain time — seconds to low minutes, and measurable in rehearsal rather than estimated.

- This is **not** dump-and-load. Worth saying explicitly, since Roofr has ruled that out and may hear "pause" as the same thing. Dump-and-load means hours of downtime proportional to database size. This means a drain of whatever accumulated in the last few seconds.
- Needs a Laravel read-only mode that degrades honestly — middleware that queues or cleanly rejects mutating routes rather than throwing 500s. Small, additive, removable.
- **Verdict:** in every version of the plan. Rehearse it, time it, quote the measured number.

### Option C — Native logical replication or binlog replication

The textbook answer: source publishes, RDS subscribes, catch up over days, verify, flip. Cheapest to operate, no code changes, clean reverse path.

- **Blocked** by everything in §4, and Roofr has already said so.
- **Verdict:** ask the one question (what exactly did Heroku say, and to whom), then treat a yes as a welcome simplification. Do not plan on it.

### Option D — AWS DMS full load + CDC

Same source-privilege dependency as C, so it is blocked for the change-stream half. **But DMS is not useless here:** its full-load capability and especially **DMS Data Validation** give row-level comparison evidence between source and target, which is exactly the proof Roofr will want before flipping. AWS will bring DMS to the table, and AWS migration funding is usually built around it.

- **Verdict:** use DMS for the initial bulk load and for validation; use Option A for the change stream. Framing it this way keeps AWS's motion intact instead of contradicting it in front of the client — which matters, because this is an AWS-originated deal.

### Option E — Application-level dual-write

The app writes to both databases through the transition.

- Every write path has to be found: raw queries, bulk operations, queue jobs. Laravel model events **do not fire** on `Model::query()->update()` or bulk inserts, so an ORM-event implementation silently misses writes. Partial-failure handling (Heroku write succeeds, AWS write fails) is genuinely hard.
- With ~70 engineers merging daily, new write paths appear during the project faster than you can instrument them.
- **Verdict:** do not propose this. It is more code change than Option A and worse odds.

### Option F — Move the database first, leave the app on Heroku

- **The trap:** Heroku Common Runtime does not support VPC peering, so that traffic crosses the public internet. A Laravel monolith amplifies every added millisecond by its queries-per-request count. Eighty queries at +3 ms is a quarter-second per page.
- Only viable on a Private Space (peering available, not on Fir-generation spaces, max 5 connections, and peered VPCs cannot reach in-space data services), and only after a per-request query census.
- **Verdict:** do not propose without latency numbers.

### The recommended shape

1. **Bring Option A + B to the call as the named mechanism**, with DMS doing bulk load and validation. That answers Kevin's actual request.
2. **A short paid spike as step one:** confirm engine and hosting, get Heroku's answer in writing, audit the schema against the §4 constraint list, and measure database size, peak write rate, GeoJSON payload sizes, and queries-per-request. It produces a go/no-go and the numbers every estimate depends on.
3. **Rehearse twice against production-sized data**, timed, with a named rollback decision-maker. Rehearsals land in Sept–Nov, during their peak — staff for that.
4. **Cut over early December as they planned.** Keep Heroku fully provisioned and warm through early January as the rollback target — already funded by the CFO's dual-infra allowance.
5. **Deprovision Heroku only after a clean January week**, not on cutover night.

---

## 6. Risks

Ranked by expected damage.

### Critical

**R1 — the GeoJSON turns out to be a native spatial column on a heterogeneous path.**
DMS maps MySQL spatial types to `BLOB`. Queries return wrong answers rather than errors, and wrong roof measurements is a customer-trust event rather than a bug ticket.
*Signal:* nobody on the call can say whether the column type is `JSON`/`LONGTEXT` or `GEOMETRY`.
*Mitigation:* keep the migration homogeneous MySQL-to-MySQL; transport spatial values as byte-exact WKB rather than re-parsed text; verify spatially — compare `ST_AsGeoJSON` output *and* WKB bytes on a sample, not just row counts. The POC does exactly this and reports both.

**R2 — A schema freeze is impossible, and the plan needs one.**
DMS does not capture several DDL forms during CDC and suspends replication on affected tables. Trigger-based capture has its own coupling to schema shape. **~70 engineers in the codebase daily means migrations ship constantly**, and a multi-week freeze across a 100-engineer org during their busiest quarter is not going to happen.
*Signal:* the plan says "freeze schema for the replication window" and nobody from Roofr has agreed to it in writing.
*Mitigation:* design for a short replication window (days, not weeks) so a freeze is survivable; add a migration-review gate rather than a blanket freeze, with a documented list of DDL that is safe versus fatal; automate a schema-drift check between source and target that fails loudly. **Raise this on the call — it is the risk their VP Eng will not have thought about, and naming it is how you earn the room.**

**R3 — No rollback once production writes land on AWS.**
Going back means losing those writes or replaying them. Teams discover this at 2 a.m.
*Mitigation:* stand up reverse replication (AWS→Heroku) before the flip, or set a hard rollback deadline stated out loud ("we can roll back for 15 minutes; after that we fix forward"). Name who has authority to call it. Their mid-January contract runway makes a real rollback option affordable — use it.

**R4 — Rehearsal window collides with peak season.**
The preparation work falls in Sept–Nov, when the platform is busiest and Roofr's engineers are least available. Under-rehearsing is what turns a good plan into a bad night.
*Mitigation:* name the required Roofr-side commitment in hours per week per role, in the SOW. Get the rehearsal dates on calendars now, in August, before their season closes in.

### High

**R5 — Sequence and auto-increment drift.** Neither Postgres sequences nor MySQL `AUTO_INCREMENT` values migrate. Scripted reset to `max(id)+buffer`, verified in rehearsal, never written on the night.

**R6 — Tables without primary keys.** DMS ignores UPDATE and DELETE on them; trigger-based capture needs a stable key too. Schema audit in the spike; add keys or full-reload those tables at cutover.

**R7 — Outbound IP allowlists break after an apparently clean cutover.** Heroku static egress (Fixie, QuotaGuard, or Private Space stable IPs) sits on partner allowlists — payment processors, insurance carriers, aerial-imagery and supplier APIs. On AWS the egress IP is a new NAT gateway address. Partner allowlist tickets run on someone else's queue.
*Mitigation:* inventory who allowlists Roofr, in the spike. Allocate Elastic IPs early. Get partners to add the new addresses **weeks** ahead, alongside the old ones.

**R8 — Inbound webhooks and OAuth redirects still point at Heroku.** Everything posting to a Heroku-routed hostname keeps working until deprovision, then stops, asynchronously and quietly.
*Mitigation:* endpoint inventory; keep Heroku routing through to AWS during the deprecation period; monitor error rates per webhook endpoint, not just in aggregate.

**R9 — SPA/API version skew at the flip.** Cached bundles calling an old origin, CORS and CSP origins not updated in step, API base URLs baked into the build.
*Mitigation:* treat the SPA as its own cutover unit with its own checklist; serve both origins during the window.

**R10 — Queue and scheduler double-execution.** Workers on both platforms against the same queue run jobs twice. Heroku Scheduler plus a new EventBridge schedule means duplicate invoices, emails, and outbound API calls. The Node PDF service is a third moving part.
*Mitigation:* an ownership matrix — for each worker and scheduled job, which platform runs it at each stage — plus idempotency on anything with external side effects.

**R11 — Connection handling changes.** Managed MySQL add-ons cap connections by plan, and JawsDB shared plans additionally cap hourly query volume. Laravel's per-request connection model plus a new pooler on AWS (RDS Proxy or ProxySQL) changes behaviour under load, and transaction-mode pooling breaks anything relying on session state, temporary tables, or `LAST_INSERT_ID()` across statements.
*Mitigation:* load-test the target at projected peak concurrency before cutover; choose pooling mode deliberately.

**R12 — Trigger write-amplification at peak.** Option A adds write cost to the source. If the measurement happens after the design is committed, it is not a measurement.
*Mitigation:* measure peak write rate and GeoJSON payload sizes in the spike; ship keys rather than payloads; be ready to exclude high-churn, low-value tables and full-reload them instead.

### Medium

**R13 — "Land flat" is a commitment we have not costed.** The CFO wants steady-state AWS spend to match Heroku and will absorb a temporary dual-infra spike. Heroku bundles a lot into its price; the AWS equivalent (RDS Multi-AZ, ALB, NAT gateway data processing, ECS/EKS, CloudWatch, Secrets Manager, DMS instances) is not automatically cheaper, and NAT gateway charges surprise people. Model this before anyone repeats "flat" back to the CFO.

**R14 — Observability gap.** Heroku's router and dyno metrics and log drains have no free AWS equivalent. Stand up comparable dashboards **before** cutover so there is a baseline to compare against, or the first AWS incident gets debugged blind.

**R15 — Config vars to secrets.** Every Heroku config var becomes a Secrets Manager or Parameter Store entry. A missing one is a boot failure; a wrong one can silently connect production to the old database. Diff the two environments programmatically.

**R16 — Ephemeral filesystem assumptions.** Heroku's filesystem is ephemeral so S3 is probably already in use, but temp-file and local-cache behaviour differs on long-lived containers. Audit `storage/`, session driver, cache driver.

**R17 — Database size still unknown.** Every duration — full load, rehearsal, drain — scales with it. Heroku's own copy method is documented at ~3 min/GB and capped at 10 GB, which shows how fast snapshot approaches stop working.

**R18 — Christmas wall plus holiday staffing.** The hard deadline sits against a period of code freezes, PTO, and skeleton on-call on both sides. A slip of two weeks from early December runs out of usable calendar.
*Mitigation:* set an internal go/no-go date in mid-November with a documented fallback (cut over in January against the contract expiry, at a known cost).

### Commercial and engagement

**R19 — Two deals are wearing one name.** The HubSpot record is "Roofr <> AI Platform (Bedrock & Anthropic)" and Camila's brief describes the need as AI strategy. The urgent, dated, technically risky work is the Heroku exit. Different scope, different team, different risk profile.
*Mitigation:* separate SOWs. Do not let the migration's December deadline and the AI workstream share a milestone.

**R20 — Fixed date against undefined scope.** Engine, size, GeoJSON type, write rate, and allowlist inventory are all open, and the deadline is fixed. That is unpriceable.
*Mitigation:* paid spike first with a named go/no-go, then a scoped migration. Do not sign fixed-scope-fixed-date before the spike returns.

**R21 — Three-party accountability.** AWS originated the deal, will advocate DMS, and may bring migration funding. When DMS hits the spatial-type limitation there will be three directions to point. Also unresolved: **Mission ran the assessment on the Dataplor pursuit, and Mike flagged TD Synnex as our route "if we are in play for this part of the opportunity"** — so who performs the migration assessment is an open commercial question, not a technical one.
*Mitigation:* a responsibility matrix — target infrastructure build, data-movement mechanism, cutover execution, rollback authority — agreed before the SOW. Settle the TD Synnex question internally before the call, not during it.

**R22 — "No PHP modernization" versus what the plan needs.** Options A and B add database objects and a read-only middleware. Establish on the call that migration scaffolding with a documented teardown is not modernization. Get that agreed early or it becomes an expensive argument mid-project.

**R23 — Roofr's bandwidth.** They own schema knowledge, the deploy pipeline, and the freeze decision, during their busiest quarter. Named hours per week per role, in the SOW.

---

## 7. The AI workstream — three distinct problems, not one

Worth separating on the call, because Roofr described them together and they need different work.

**Attribution and cost visibility.** Bedrock **application inference profiles** are the direct answer: a tagged profile per team or workstream, invoked by ARN, with usage landing under AWS cost allocation tags. Single-Region or wrapping a cross-Region profile. This is demoable and it precisely matches what Kevin described as missing.

**Guardrails for non-engineers.** People defaulting to frontier models with no controls is an access-and-routing problem — who can reach which model through what interface — not only a Bedrock Guardrails content-filtering problem. Amazon Q came up alongside Bedrock, which suggests the internal-user surface matters as much as the API. Scope what non-engineers are actually using today before proposing the control layer.

**Losing DX tracking data at merge.** PRs to main dropping developer-context data is an engineering-metrics pipeline problem. It has nothing to do with Bedrock routing and should not be bundled into the same workstream, or it will be the thing that makes the AI engagement look like it under-delivered.

Risks specific to this workstream:

- **Bedrock is not automatically more headroom.** It has its own per-model, per-Region quotas. Scaling means quota increases, cross-Region inference, or provisioned throughput. Do not let "route through Bedrock" be heard as "capacity problem solved."
- **Model and feature parity lag.** Bedrock does not always expose the newest Anthropic models or every API capability immediately. Check per model, per Region against what Roofr uses today.
- **Guardrails add per-call latency and cost** and can block legitimate content. Measure before putting them in a user-facing path.
- **Routing does not reduce unit price.** If the CFO expects a smaller bill rather than better visibility, the levers are model selection, prompt caching, and batch. Separate "we want attribution" from "we want to spend less" now — different work, different deliverable.
- **Client-side migration effort.** Anthropic SDK to Bedrock means IAM instead of API keys, changed request shape, changed error handling. In a monolith with no modernization budget, scope it as a thin adapter behind one interface.

---

## 8. Open questions

Ordered by how much they unblock. The first two are answerable by their new AI lead in a single message before the call — worth asking now rather than spending call time on them.

**Blocking the design**

1. **Who hosts your MySQL today — JawsDB, ClearDB, or somewhere else? Which plan?** If it is JawsDB it already runs on AWS, which could materially shorten the database half of this project.
2. Is the GeoJSON in a `JSON`/`LONGTEXT` column or a native MySQL spatial column? Do you run spatial queries and spatial indexes in the database? How large are the largest payloads?
3. Database size, peak write rate, and rows-per-second on the heaviest tables. **Backfill wall-clock is a direct function of these and it is the single biggest input to the schedule.**
4. Exactly what did you ask your database provider about replication and binlog access, and what did they say? Who asked, and how recently?

**Platform**

5. Common Runtime or Private Space? Which region?
6. Full add-on inventory: Redis, Scheduler, log drains, static-egress add-ons.
7. Who allowlists your outbound IPs, and who posts webhooks to you?
8. Queue driver, session driver, cache driver, and what the scheduler runs. Where does the SPA get served from, and where does the Node PDF service run?
9. Any table without a primary key? Any `enum`, `range`, or composite columns? Materialized views?

**Scope and constraints**

10. What does zero downtime mean to you — no failed requests, or no data loss with a brief write-pause? Is a pause of under a minute acceptable? Under five?
11. Can you gate schema migrations for a short window? How short is tolerable across 70 daily contributors, and who enforces it?
12. What did December look like in each of the last three years, in traffic and write volume? Does post-hurricane claim work extend your peak past November?
13. Does "no PHP modernization" permit additive, removable migration scaffolding — triggers, a change-log table, a read-only middleware?
14. Who has authority to call a rollback at 2 a.m., and what is the threshold?
15. What target AWS architecture have you and AWS discussed — ECS, EKS, EC2, RDS or Aurora, Multi-AZ?
16. Is AWS migration funding (MAP or equivalent) in play, and what does it require of the delivery plan?

---

## 9. What to bring into the room

- **The mechanism, first.** Option A + B, named, with the DMS bulk-load-and-validate split. Kevin asked for specifics and said so twice. Lead with the trigger-based approach and the reason it works when replication does not.
- **The evidence that we understand their constraint.** The §4 documentation trail shows we independently verified "no replication support on Heroku" rather than nodding along. Short version, not the full list.
- **The GeoJSON question.** If nobody in the room can say `geometry` or `jsonb`, you have just shown exactly what a sales engineer is for. Ask their AI lead beforehand if you can.
- **R2, the schema-freeze problem.** Seventy engineers merging daily against a replication window nobody has costed. This is the risk their VP Engineering has not priced, and naming it early is the fastest route to technical credibility.
- **The rollback runway.** Heroku is contracted through mid-January and the CFO has funded the dual-infra overlap. Cutting over in early December with Heroku warm until January turns their headline fear into a bounded one. Their champion needs this argument for the room we are not in.
- **The Dataplor precedent.** Kevin asked for prior client examples. Mike posted the Dataplor Heroku migration SOW and the Assess Findings / Migrate Plan in #roofr-dealteam-sales (Slack files `F0BQCKU3NC8`, `F0BPJ8ZKSRY`) — they are not in the Drive folder. Pull them, sanitize, and decide what is shareable before the call. Also settle internally whether the assessment runs through TD Synnex.
- **The two-SOW framing.** Migration and AI governance are separate engagements that happen to share a client. Say it before someone bundles them into one December milestone.

---

## Sources

**Internal**

- #roofr-dealteam-sales (Slack, C0BPG0Z1M2A) — Blaine Wagner's deal brief 11 Aug; Camila Lopez's HubSpot deal post; Mike Dennis's Dataplor artifacts and TD Synnex note.
- [Roofr Meeting Notes 8/11](https://docs.google.com/document/d/13A3iuChF65fSjkYWYi70b4dh0GSMFFqnzm-KybsGlIU/edit) — intro call, 11 Aug 2026 (Kevin Prince / Roofr, AWS, Mike Dennis). Transcript linked in the doc.
- [Roofr - Sales Drive folder](https://drive.google.com/drive/folders/17PSpG3xxEasEjF3LHga7Je6Vjm2I8ziq)
- [computer://Users/rodrigo/Dualboot/Tolaria/Personal/roofr.md](computer://Users/rodrigo/Dualboot/Tolaria/Personal/roofr.md) — the Tolaria note, rewritten 18 Aug 2026 and now the accurate internal summary. Its LAMP/MySQL claim is confirmed; its earlier "manufacturing technology company" line came from the HubSpot Industry field and has been removed.
- `engagements/roofr-heroku-exit/poc/` — the working proof of concept and its run report. MySQL, backfill plus change stream, 245 ms measured write pause, zero divergence.

**Vendor documentation — MySQL, their stack**

- [AWS DMS — MySQL as a source](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.MySQL.html) — binlog requirements (`ROW`, `FULL`, retention), `REPLICATION SLAVE`/`REPLICATION CLIENT` grants, spatial types mapped to BLOB, `AUTO_INCREMENT` not migrated, `GENERATED` column changes not captured, partial DDL support during CDC, 4 GB binlog limit.
- [JawsDB](https://devcenter.heroku.com/articles/jawsdb) — hosted on AWS; restricted privileges; SUPER limitations; hourly query caps on shared plans; no documented binlog access or external replication.
- [Private Space VPC Peering](https://devcenter.heroku.com/articles/private-space-peering) — Common Runtime unsupported; Fir-generation unsupported; peered VPCs cannot reach in-space data services. Relevant to Option F regardless of engine.
- [Bedrock application inference profiles](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-create.html) — per-profile usage/cost tracking, tagging for cost allocation.

**Vendor documentation — Postgres, retained for context only**

These informed revision 2, when the engine was still open. Kept because the POC still ships an optional Postgres path and because the contrast is occasionally useful, but none of it describes Roofr's stack.

- [AWS DMS — PostgreSQL as a source](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.PostgreSQL.html) — PostGIS homogeneous-only; CDC privilege prerequisites.
- [Heroku Postgres Extensions](https://devcenter.heroku.com/articles/heroku-postgres-extensions) — pglogical/wal2json/test_decoding absent.
- [Heroku Postgres Follower Databases](https://devcenter.heroku.com/articles/heroku-postgres-follower-databases) — physical, Heroku-only, no external followers.
- [Upgrading Heroku Postgres Databases](https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases) — documented downtime figures.