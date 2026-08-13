---
type: Note
belongs_to: "[[wg-henschen]]"
related_to: "[[wg-henschen-rag-scope-call-notes]]"
status: Active
_width: wide
---

# WG Henschen — NetSuite Integration Options

Research pass on how we can actually get data out of NetSuite and enforce permissions, ahead of estimating. Sources are Oracle NetSuite Help Center and AWS documentation, checked 13 Aug 2026. Where the docs don't say something, it's marked unverified — those are the items to test in a sandbox, not to assume.

Companion to [[wg-henschen-rag-scope-call-notes]].

## The one-line version

Two findings change the shape of the estimate.

**Good news:** SuiteQL enforces the querying role's data restrictions. Oracle states it directly — *"SuiteQL enforces the same role-based access restrictions used in SuiteAnalytics Workbook."* If we route reads through SuiteQL under a per-user token, the permission layer we sketched on the call largely comes for free instead of being rebuilt. That was going to be one of the more expensive components.

**Bad news:** there is no NetSuite connector in Bedrock Knowledge Bases, Amazon AppFlow, or Amazon Q Business. The only native AWS-side NetSuite integration is an AWS Glue connection, and it has a hard 100,000-record extract cap plus documented unreliable date filters on exactly the objects we need (Item, Transaction Line, Transaction Accounting Line). Extraction and sync are custom work. Nobody should be quoting this as "wire up the connector."

## What we don't know about their instance — and it drives everything

I couldn't research WG Henschen's specific NetSuite footprint. Web search was blocked in the session, and their configuration isn't public anyway. But the variables below aren't nice-to-haves — each one moves the architecture, so they belong at the top of the questions to David.

| Unknown | Why it matters |
|---|---|
| **Service tier** (Standard / Premium / Enterprise / Ultimate) | Sets the account concurrency limit: 5 / 15 / 20 / 20. On Standard that is **5 concurrent API requests for the entire account**, shared with every other integration they run. |
| **SuiteCloud Plus licenses** | Each adds +10 concurrency. Visible at Setup > Integration > Integration Management > Integration Governance. |
| **Is SuiteAnalytics Connect licensed?** | Separately purchased add-on. Having it removes the 100k row ceiling on bulk extract and raises REST paging from 100k to 1M. Not available at all on NetSuite Small Business. |
| **OneWorld?** | Determines whether subsidiary restrictions are in play — a whole axis of the permission model. |
| **Which modules beyond inventory** | Advanced Inventory, WMS, Manufacturing/Work Orders, Demand Planning, Quality Management, SuiteBilling, Fixed Assets, CRM. Aerospace normally means serialized/lot traceability and AS9100-driven records, which is a different data shape than plain distribution. |
| **Custom records and custom fields** | Aerospace part masters are almost never stock NetSuite. Custom fields surface as `custbody<n>`, custom records as `CUSTOMRECORD<n>`. Volume here is a direct driver of data-modelling effort. |
| **NetSuite Analytics Warehouse (NSAW)?** | If they have it, there's a modelled warehouse already. But it refreshes every 24 hours — D-1 data, wrong for "how many of this part do we have right now." |
| **Existing integrations** | Anything already consuming the concurrency pool. |
| **ITAR / export control on part data** | Aerospace. If any part data is export-controlled, that constrains region, model selection, and data retention before anything else does. Not raised on the call and it should have been. |

## Extraction surfaces — what we can actually use

| Surface | Fit | Ceiling | Extra SKU |
|---|---|---|---|
| **SuiteQL over REST** | **Primary path** | 100,000 rows/query (1M with Connect) | No |
| SuiteAnalytics Connect (ODBC/JDBC) | Best for initial backfill | No documented row cap | **Yes** |
| REST record endpoints | Targeted record reads | 100,000 per collection | No |
| RESTlets | Escape hatch | 5,000 usage units, 300s | No |
| NSAW | Historical/aggregate only | — | Yes (effectively) |
| Saved search email export | Not viable | **10,000 rows / 5 MB, silently truncated** | No |
| SOAP | **Do not build on it** | Disabled at 2028.2 | No |

### SuiteQL over REST — the recommended primary path

`POST /services/rest/query/v1/suiteql` with a `Prefer: transient` header. Arbitrary SQL against the NetSuite2.com analytics data source: joins, aggregates, GROUP BY. Same SuiteQL dialect is available through SuiteAnalytics Connect and the `N/query` SuiteScript module, so query logic is portable between them.

Constraints worth designing around:

- 100,000 results maximum per query. Every query must be windowed — no assuming a full-table pull.
- Oracle SQL syntax is strongly recommended; ANSI SQL-92 is supported but Oracle's own docs warn it *"may cause critical performance issues, timeouts, and conversion problems."* Cannot mix the two dialects in one query.
- Max 1,000 arguments in an `IN` clause. No async execution on the query service.
- Field-name casing is documented as inconsistent. Don't rely on it.
- Sorting on CLOB fields only evaluates the first 250 characters.

### SOAP is on a removal timeline

Worth knowing in case they have existing SOAP integrations that need to move: no new SOAP integrations from 2027.1, TBA support for SOAP ends 2027.1, all SOAP endpoints disabled at **2028.2**. If WG Henschen has anything on SOAP today, that's a separate conversation — and possibly a second line of business.

One capability gap in the migration: SOAP's `getDeleted` operation has no clean REST equivalent. For tombstone detection we'd use the `deleted_records` table via SuiteAnalytics Connect, or accept a gap.

### Concurrency is the constraint people miss

Since 2017.2, REST + SOAP + RESTlet requests all draw from **one shared account-level pool**. Formula: `base tier limit + (10 × SuiteCloud Plus licenses)`.

On a Standard tier with no SuiteCloud Plus that is 5 concurrent requests, total, across every integration they own. A sync job that spins up 20 workers will start returning HTTP 429 `CONCURRENCY_LIMIT_EXCEEDED` and will also break whatever else is talking to their NetSuite. Oracle's own reference retry loop uses 5 attempts with a wait.

Notably there is **no documented requests-per-day or requests-per-second rate limit** — governance is concurrency-based, not rate-based. Anyone quoting a "NetSuite daily API limit" is repeating folklore.

### Incremental sync — the weakest-documented area and the biggest delivery risk

What Oracle does document:

- `last_modified_date` / `date_last_modified` as the watermark column, and it's indexed — this is the supported incremental pattern.
- `deleted_records` table (Connect / analytics data source) for tombstones.
- And the caveat that matters: *"Some tables do not support `last_modified_date`, `date_last_modified`, and `deleted_records`, for example, mapping tables. You must download all data from these tables every time."*

So the sync design is a hybrid: watermark most tables, tombstone-poll for deletes, full-reload the mapping tables every cycle. That last category is unbounded until we see their actual schema — which is why an access-first discovery spike matters more than a spreadsheet of hours.

## Permissions — better than expected

This was the piece I'd have guessed was expensive. It mostly isn't, if we build it the right way.

**The model.** NetSuite separates *permissions* (can this role touch this record type, at NONE/VIEW/CREATE/EDIT/FULL) from *restrictions* (which instances of that type). Row-level restriction is first-class and declarative, on five axes: employee, department, class, location, and subsidiary (OneWorld only). Field-level access control exists for **custom** fields via the Access subtab (Edit / View / Run / None), resolved as highest-access-wins across role, department and subsidiary. I found no documented equivalent for standard fields — flag as unverified.

**Why this matters.** SuiteQL is documented as enforcing exactly these restrictions. A query issued under a user's role returns only what that user could see in SuiteAnalytics Workbook. We inherit the client's existing access model rather than reimplementing it — and more importantly, we don't become the team that accidentally leaks margin data to a warehouse role.

**The one thing that would destroy this.** SuiteScript deployments have an **Execute as Role** field. Set it to a fixed privileged role and every query runs as that role regardless of who asked. Same failure mode with OAuth 2.0 **client credentials**, which is explicitly bound to one fixed entity+role pair — a service account, not an end user. Either choice silently collapses the entire permission story into "everyone sees everything." This needs to be a written architectural constraint, not tribal knowledge: **authorization code flow or in-session current-user context, never client credentials, for anything user-facing.**

**Identity to our AWS backend.** Use **NetSuite as OIDC Provider** — NetSuite acts as the OpenID Provider, discovery at `/.well-known/openid-configuration`, RS256-signed ID token valid 3 hours, `sub` claim carries `role;entity`. That is the documented way for an external service to verify who the NetSuite user is. There is no documented API for in-session code to mint a NetSuite-signed user assertion, so the alternative would be a trust construction of our own design — avoid it.

**Auth housekeeping:** access tokens 60 min, refresh tokens 7 days (2 days and single-use for public clients). Don't build on TBA — no new TBA integrations for REST/SOAP/RESTlets as of 2027.1.

## Embedding the chat UI — use the SPA script type

The call assumed "likely a React embed." There's a better-defined answer than a hand-rolled Suitelet.

NetSuite has a first-class **Single Page Application** script type, built on the User Interface Framework, with React support through the UIF library, deployed via SDF as a SuiteApp. Concretely it gives us:

- A **Release Audience** setting that gates access by role, verified by the server script at load.
- **Center Links** to surface it in NetSuite navigation.
- And the sentence that settles the permission question: *"The SPA always executes in the browser based on the currently logged-in user's role, regardless of the 'Execute As' setting."*

Requires SuiteScript 2.1, an SDF SuiteApp project, and a build chain that transpiles JSX and converts ESM to AMD (RequireJS). Oracle publishes samples with a Gulp setup. SuiteApp packaging is not optional on this route — factor SDF project setup and SuiteApp Control Center distribution into the estimate.

Fallback is a Suitelet serving raw HTML via `response.write` with bundles in the File Cabinet. Workable, but we'd be rebuilding the audience gate and navigation ourselves.

**Open risk to test cheaply and early:** I found no documented Content Security Policy or external-domain allowlist in the NetSuite help — but absence of documentation is not absence of enforcement. Before committing to loading bundles from our own CDN or embedding a cross-origin iframe, check the actual `Content-Security-Policy` and `X-Frame-Options` response headers on a rendered SPA page in a sandbox. Half a day of work that de-risks the whole front-end approach.

Also: anything placed in Web Site Hosting Files is publicly fetchable by default. Bundles there, never data.

## AWS side

**No native NetSuite connector anywhere that matters.** Bedrock Knowledge Bases connectors are S3, SharePoint, Confluence, Salesforce, Google Drive, OneDrive, Web Crawler, and custom. AppFlow's 79 connectors include Oracle HCM but not NetSuite, and its generic JDBC connector only speaks MySQL and PostgreSQL. Amazon Q Business has no NetSuite connector and is closed to new customers.

**The one native bridge is AWS Glue's "Oracle NetSuite" connection.** Read-only, SuiteTalk REST v1. Documented limitations that bear directly on us:

- **Maximum 100,000 records per extract.**
- On **Item, Transaction Line, and Transaction Accounting Line**, date filter operators are unreliable: `EQUAL_TO`/`NOT_EQUAL_TO` unreliable, `LESS_THAN_OR_EQUAL_TO` behaves as `LESS_THAN`, `GREATER_THAN` behaves as `GREATER_THAN_OR_EQUAL_TO`.

Those are precisely the objects an inventory-and-accounting assistant lives on, and precisely the filters incremental sync depends on. Mitigation is over-fetching a window and deduping — cost, not blocker, but it needs to be in the number. Glue zero-ETL does not support NetSuite either.

**Structured data matters more than documents here.** Bedrock Knowledge Bases supports natural-language-to-SQL over structured data, which is the right shape for "how many of part X do we have" — those are aggregate queries, not document lookups. Caveats:

- **Amazon Redshift is the only query engine** (native Redshift or Glue Data Catalog through Redshift).
- **Maximum 10 tables per knowledge base, 100 columns per table, 60-second query timeout.** A NetSuite replica has hundreds of tables. We must design a curated mart of ≤10 wide tables. That is a real modelling exercise and one of the larger line items in the estimate — and it's a decision that has to be made *with* the client, because it defines what the assistant can and cannot answer.
- Table/column descriptions and curated NL/SQL example pairs are the accuracy levers.
- Oracle's own note: include/exclude lists are "non-deterministic and intended for improving accuracy, not security." Not a permission mechanism.

**The "I don't have that information" requirement.** Bedrock Guardrails supports it natively via **contextual grounding checks** — separate grounding and relevance scores, thresholds configurable 0 to 0.99, plus a configurable blocked-message string that becomes the "I don't have that information" text. Denied topics and PII filters layer on top.

One caveat to raise with the client rather than discover in UAT: the docs state contextual grounding is supported for summarization, paraphrasing and question answering, and explicitly that **"Conversational QA / Chatbot use cases are not supported."** Limits are 100,000 characters of grounding source, 1,000-character query, 5,000-character response. A multi-turn chat assistant is exactly the shape they described, so we should plan on per-turn stateless grounding evaluation and set expectations accordingly. There's also a streaming caveat: an irrelevant response may fully stream before being marked irrelevant.

**Models and data handling.** Claude Opus 5 and Sonnet 5 are available on Bedrock. Use a **geography-scoped inference profile** (`us.` / `eu.`) rather than `global.` for data residency. Bedrock documents a `data_retention_mode` with a `none` (zero data retention) option — worth pinning explicitly given the client's sensitivity, and worth verifying per model ID, since some newer models require a provider-data-share mode retaining prompts up to 30 days. Given the prior engagement's whole rationale was keeping patented drawings inside a controlled boundary, this should be an explicit written commitment in the proposal, not an implementation detail.

## Recommended architecture, with the trade-off named

| Question type | Path |
|---|---|
| Aggregate / numeric — stock on hand, AR aging, GL balances | Curated Redshift mart → structured-data KB, NL2SQL |
| Document lookup — SOPs, contracts, certs, attachments | S3 → vector KB |
| Live point lookup — "status of PO 12345 right now" | Agent tool calling SuiteQL through Lambda |
| Refusal behaviour | Guardrails contextual grounding + denied topics + custom blocked message |

The open architectural decision is **replicate vs. query live**, and it should be presented to the client as a trade-off rather than decided for them:

- **Replicate to Redshift.** Fast, cheap per query, supports aggregates and the NL2SQL knowledge base, survives NetSuite's concurrency limit. Costs data freshness, and it **breaks the free permission inheritance** — once data is in Redshift, NetSuite's role restrictions no longer apply and we have to reimplement them.
- **Query NetSuite live via SuiteQL.** Always current, permissions enforced for free. Costs latency, and it consumes the shared concurrency pool that their other integrations depend on.

A hybrid is probably right — replicate the non-sensitive, aggregate-heavy inventory data; query live for anything permission-sensitive or point-in-time. But that's a choice with cost and risk attached, and the client should make it knowingly.

## What to confirm before the estimate is real

Ordered by how much each one moves the number.

1. **Service tier, SuiteCloud Plus license count, and whether SuiteAnalytics Connect is licensed.** Visible in one screen at Setup > Integration > Integration Management > Integration Governance. Everything about throughput follows from it.
2. **A sandbox account with an integration role.** Not production. Half a day in a sandbox answers more than three more calls — schema shape, custom record volume, whether CSP blocks our asset strategy, whether REST record responses respect restrictions.
3. **Who owns NetSuite on their side.** Still unidentified, and it's the critical-path dependency. Internal admin, or an outside NetSuite partner? Every question above needs someone who can answer it.
4. **Which modules and how much customization.** Drives the mart design against a hard 10-table ceiling.
5. **Export control status of part data.** Aerospace. Ask before designing anything that moves data.
6. **Chat-only or exportable output?** Raised on the call, still open. Adds a deliverable if yes.

## Verified-vs-unverified

Everything above with a number attached is from Oracle or AWS documentation. Explicitly **not** verified, and worth testing rather than trusting:

- Whether REST *record* endpoints (as opposed to SuiteQL) filter by role restrictions. The role binding is documented; the filtering behaviour is not stated in words.
- Whether NetSuite2.com is a live view or a replica, and any replication lag.
- SuiteAnalytics Connect concurrent-connection limit and query timeout.
- Whether NetSuite enforces a CSP or frame-ancestors policy on Suitelet/SPA pages.
- Any documented field-level security for standard (non-custom) fields.
- OAuth 2.0's concurrency treatment — TBA is documented, OAuth 2.0 is not named.
