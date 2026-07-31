---
type: Note
related_to: "[[artsavl]]"
status: Active
---

# ArtsAVL — Solution Architecture (Schema-per-Tenant)

Draft for proposal, 2026-07-31. Two decisions directed by Rodrigo rather than left open: **schema-per-tenant** as the tenancy model, and **all six goals delivered thin inside six months** as the bid shape. Grounded in [[artsavl-technical-assessment]]; answers [[artsavl-risk-register]].

**Not validated against source code.** Roughly a third of this is conditional on answers ArtsAVL has not given. The delivery model is built so those answers arrive at gates rather than as surprises.

## The plain-language version

ArtsAVL owns a working, genuinely current platform. Keep it and extend it — nothing in the evidence supports a rebuild. Five things happen:

1. **Take custody properly.** Verified access to source, hosting, payments, email and every dependent service, with a live test payment and test email proving each path before anything changes.
2. **Find out what members and staff actually need**, rather than guessing from a feature list, and write it down so ArtsAVL owns it.
3. **Improve the everyday things** — being found in search, joining and renewing, posting an event, selling an advert, moderating submissions, seeing what works.
4. **Give each partner arts council its own private space inside one system** — their own web address, look, members and local categories, while ArtsAVL maintains one platform rather than one per partner. Each partner's information is stored separately from every other partner's, which is the part their boards will ask about.
5. **Widen distribution** so listings reach news outlets, tourism sites, libraries and community calendars without anyone retyping them.

## Architecture

**One Rails application on Heroku, one codebase, one deploy.** Request flow: hostname → tenant resolution → PostgreSQL `search_path` set for that tenant for the life of the request → normal Rails handling, unchanged.

**One PostgreSQL database, many schemas.**

- `public` — tenant registry, global user credentials and roles, ActiveStorage blob metadata with tenant attribution, GoodJob queue, shared member-attribute vocabulary, cross-tenant rollup tables.
- `tenant_artsavl`, `tenant_<partner>` — profiles, events, opportunities, memberships, advertisements, impressions and clicks, moderation history and audit trail, per-tenant taxonomies including regions and categories.

**Implementation deliberately boring:** a thin request middleware and job wrapper setting `search_path`, **not a multi-tenancy gem**. Two reasons — §12 requires disclosing third-party components affecting ArtsAVL's control of its own platform, and a client with no engineers should not inherit a dependency whose maintenance status they cannot assess. Roughly a hundred readable lines.

### Why schema-per-tenant, and what it costs

Physical namespace separation is the isolation story a partner council's board will ask about, and ArtsAVL can state it plainly. A departing partner is a single schema dump — a clean exit, which matters to organizations that just watched ArtsAVL change vendors. And unlike database-per-tenant, `search_path` switching runs on the **existing shared connection pool**, so connection count does not grow with partner count.

Three things get harder:

1. **Migrations run once per schema**, with partial-failure handling. Scripted at 3–5 partners; a system of its own at fifty.
2. **Cross-tenant queries become expensive.** This is the significant tension, because **Goal 5's syndication ambition is inherently cross-tenant.** Answered by denormalized **rollup tables in `public`, written by GoodJob** — derived, rebuildable data only, so no partner's system of record leaves their schema. Trade-off is eventual consistency: a cross-partner feed may lag a partner's own site by minutes. Acceptable for arts listings, and better said than implied.
3. **The test suite grows a new class of assertion** — cross-tenant leakage tests in the inherited 2,000-example suite. Any query reaching a tenant table outside a resolved tenant context raises rather than returns.

### Other core decisions

- **Identity:** global credentials in `public`, membership and roles per tenant. An artist exhibiting in two counties should not maintain two logins. Authorization becomes explicitly two-part.
- **Payments:** Stripe hosted Checkout stays. **v1 keeps ArtsAVL as merchant of record**, remitting to partners; Connect deferred as a priced option (risk T1).
- **Tenant resolution:** hostname-based — subdomain by default, custom partner domain supported. Custom domains are what partners actually want; each adds certificate cost, DNS coordination with an organization that may also have no technical staff, and a support interaction.
- **Per-partner branding — a consequence worth knowing before pitching licensing.** Bulma themes compile from Sass variables, so recompiling per tenant would require exactly the JavaScript build step the platform deliberately avoids. Branding is therefore delivered as **runtime CSS custom properties** — colour, logo, typography, header imagery — layered over one compiled stylesheet, with contrast validation so partners cannot brand themselves into an accessibility failure. **It does not cover per-partner layout differences.** Deeper theming requires renegotiating the no-build-step constraint first.
- **Discoverability:** server-render the first page of listings, generate per-tenant sitemaps, add schema.org structured data, add keyword search to the directory. The same structured-data work serves Goal 1 and Goal 5.
- **Syndication:** leave the existing JSON event endpoint contract **untouched** — its consumers are unidentified, so any change is potentially breaking for a partner nobody has mentioned. Add alongside: per-tenant iCal feeds (what community calendars actually ingest), JSON Feed, an embeddable **iframe** widget (no build step, no host-page compatibility work), per-consumer keys with rate limiting so ArtsAVL can see who consumes what — a prerequisite for ever charging for it.
- **Reporting:** dashboards read application data via rollups. Google Analytics cannot report on members, renewals or advertising performance because those are application entities.
- **Communications and CRM:** integrate, build nothing. SendGrid keeps transactional email; newsletters stay in whatever platform serves the 7,000-subscriber lists with automated member→subscriber sync; Little Green Light keeps donor records with an identity link. Both blocked on identification.
- **Framework currency:** Ruby 4.0.6 patch in Phase 0. **Rails 8.1 leaves active support 2026-10-10, inside the engagement** — the 8.2 upgrade belongs in the first annual maintenance release. Pricing maintenance without it prices a known dated event as a surprise.

## Sequence

1. Transition and continuity — verified access per service, test transaction and test send, Ruby patch, legacy domain redirect. Everything depends on it.
2. Assessment, bounded UX discovery, accessibility audit, tenancy spike, peer-council validation, baseline capture → **re-baselining decision point**.
3. Discoverability and structured data — deliberately early; serves two goals with one piece of work and is the most visible early proof to members.
4. Tenancy foundation → 5. per-partner configuration and branding → 6. rollups and reporting → 7. syndication surface.
8. Revenue workflow improvements (self-service ad purchase, nonprofit package out of email, renewal recovery).
9. Integrations — blocked on newsletter platform and LGL identification.
10. First partner onboarding.

**The dependency that matters:** the tenancy foundation gates items 5, 6 and 10, so a delay in the Phase 1 spike propagates to more than half the plan. Items 3 and 8 sit deliberately outside the tenancy chain — **they are what still ships if tenancy slips**, and that is the contingency the thin scope needs.

## What "thin" means

The full in/out lines per goal live in the engagement document. The one to read is **Goal 4's exclusions**: no Stripe Connect or per-partner payment collection, no partner self-service signup or admin portal, no per-partner layout differences, no partner support desk, **one partner onboarded rather than several**, no cross-partner unified search.

Also excluded across the plan: full visual redesign, accessibility remediation beyond the top tier, segmentation or campaign tooling built in-platform, new pricing or tier design (ArtsAVL's decision), bespoke per-outlet syndication integrations, 24/7 on-call, a formal uptime SLA.

**If these exclusions are traded away in negotiation, the plan has no slack left.**

## Delivery and team

Phase 0 short and firm. Phase 1 discovery, audits, spike and validation in the first six weeks. Then two-week increments with a written decision record at each boundary.

**Dualboot:** product lead who also carries the technical narrative to Katie and the board, project manager, Rails technical lead, Rails engineers, UX designer through discovery and the enumerated Goal 1 work, QA against the inherited suite. Product lead and technical lead **committed by name** across the maintenance term — that is what RFP §8's continuity question is asking.

**ArtsAVL — this determines the real pace, and belongs in the proposal as a stated assumption:**

- Executive sponsor (Katie Cornell) — gate decisions, the three validation conversations, board communication.
- **A single named product owner at 8–10 hours per week sustained.** Probably Maria Buchanan. The assumption most likely to break; the proposal must state what slips if it does — discovery lengthens, acceptance queues, the tenancy gate moves.
- Staff SMEs for moderation, advertising, reporting and programs.
- A board-side interpreter — Brandy Bourne is the best available candidate, unverified.

**Governance:** a written decision record in ArtsAVL's possession at every gate. With no internal technical counterpart, the written record *is* the institutional memory — and it is what preserves ArtsAVL's ability to leave us, which §12 says they intend.

## Open options

Partner addressing (subdomain vs custom domain) · taxonomy scope (fully per-partner vs shared category spine with per-partner regions) · partner payment collection (remit vs Connect — **close before signature, not during delivery**) · theming depth (CSS custom properties vs compiled themes plus a build step) · cross-partner experience (rollup feeds only vs genuine regional search) · maintenance release content (patches only vs including the Rails 8.2 upgrade).

## What would change this

Repository access · whether partners must collect payments directly in year one · how many partners ArtsAVL wants live in year one · what varies per partner · whether the no-build-step constraint holds · newsletter platform and LGL identity · Linkup and Rastrac · the WCAG conformance level expected.
