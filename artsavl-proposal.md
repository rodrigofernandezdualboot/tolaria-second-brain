---
type: Note
related_to: "[[artsavl]]"
status: Draft
_width: wide
---

# ArtsAVL Proposal

**Draft for the 2026 Creative Directory & Engagement Platform RFP.** Due 2026-08-15. Built from [[artsavl-technical-assessment]], [[artsavl-solution-architecture]], [[artsavl-risk-register]], [[artsavl-estimation]] and the live-site crawl. Sections are ordered to cover every area ArtsAVL said it would assess, without referring to its scoring scheme.

> **Before this can be sent — items only Dualboot can supply.** Everything else below is drawn from evidence. These are marked `[NEEDS INPUT]` inline and deliberately left blank rather than filled with plausible-sounding text:
> - Three to five reference engagements with client names, dates, our role and current status
> - Three referenceable clients with contact details for long-term product/support engagements
> - Named team members for product lead, project manager, tech lead, UX
> - Firm overview: ownership, size, office locations
> - Blended rate and the resulting fee figures
> - Whether we bid the phased scope (recommended, fits budget) or all six goals at ~37% over
>
> **Unresolved commercial decision.** The estimate puts all-six-thin at ~2,743 hours against a budget that buys 1,650–2,000. This draft is written around the **phased** shape because that is what fits and what §11 invites. If we bid all six, §3 and §7 change materially.

---

## 1. Our understanding of ArtsAVL and the Creative Portal

ArtsAVL has been Buncombe County's designated arts agency since 1952, and the Creative Portal is not a marketing site — it is a working business platform that carries part of the organization's unrestricted revenue. Membership at $75 a year, a $250 nonprofit sponsorship package, and $100-a-month web advertising are real income against a real cost base, and they sit alongside two newsletters reaching more than 7,000 subscribers at an open rate above 60%. That combination — public discovery on one side, a secure self-service member dashboard on the other, with staff review in between — is what distinguishes the platform from an ordinary directory, and it is why the RFP is right that this is a product rather than a website.

We also read the RFP as arriving at a specific moment. The platform has just been through a deliberate modernization and documentation effort in preparation for handing it to a new partner: current Ruby and Rails, more than 2,000 automated examples, 154 end-to-end journey tests, a guarded deploy pipeline, a maintained conventions guide and API reference. That is an unusually well-prepared handoff, and the RFP is explicit that none of it should be re-billed. Our reading is that ArtsAVL is not looking to start over. It is looking for someone to take custody responsibly and then build.

Three things we think matter more than they appear in the document.

**The platform's revenue base is larger than the price point suggests.** Directory profile records run into the thousands. At $75 a year, membership is plausibly a six-figure annual line — which makes conversion, renewal and the everyday experience of joining and maintaining a profile a material financial lever, not a cosmetic one. We would want the real figures early, because they change which improvements pay for themselves fastest.

**Licensing is an organizational change, not a feature.** Making the platform available to other arts councils under their own branding turns ArtsAVL into a software provider: partner onboarding, per-partner configuration, questions arriving from other organizations' staff, and release management across more than one live instance. The engineering is the smaller half of that. We have priced and sequenced the engineering, and we flag the operating half plainly, because a five-person team taking on a support obligation is a real commitment that should be made deliberately.

**The current taxonomy is local by design.** The directory filters on fourteen geographic regions that are Asheville neighbourhoods and Buncombe County towns — Downtown, West Asheville, the River Arts District, Black Mountain, Fairview, Weaverville. A partner council in another county needs its own regions, while ArtsAVL will want enough shared structure that cross-partner search and syndication still make sense. That tension between local and shared is the central design question in the licensing work, and we would rather name it now than discover it during build.

**Two things we would want to confirm early.** First, the RFP describes reporting, automated and digest communications, and activity tracking as existing capabilities, and we take that at face value — but the extent of what exists determines how much of the reporting and engagement goal is enhancement versus new construction, and that is one of the larger unknowns in our estimate. Second, the integration list in §4 includes two services whose role we could not determine from outside; we would resolve that during onboarding rather than assume anything about them.

## 2. Relevant experience

`[NEEDS INPUT — three to five engagements, with client name, description, our role, dates, current status. The RFP asks specifically for existing Rails applications, membership or directory products, advertising or revenue-generating platforms, multi-tenant or white-label products, and long-term support relationships. Do not pad this list; it is weighted heavily and a thin entry is worse than a shorter list.]`

`[NEEDS INPUT — three referenceable clients with contact details, for comparable long-term product development or application-support engagements.]`

What we would want this section to demonstrate, in order of importance to ArtsAVL: that we maintain and extend mature Rails applications as a normal part of our work rather than as an exception; that we have taken over someone else's codebase before and can describe how that went; that we have built membership, directory or community-engagement products where the revenue mattered; and that we have kept a client for years rather than months.

## 3. How we would approach the work

### Onboarding and continuity, first

Nothing else is safe until custody is established. Before any feature work, we verify our own access and ArtsAVL's ownership across every service the platform depends on — source control, hosting, payments, email, storage, caching, bot protection, monitoring, mapping and analytics — and we prove the critical paths still work by putting a live test transaction through payments and a test message through email delivery. We would also confirm that accounts and billing sit in ArtsAVL's name rather than a previous partner's, because ownership of the accounts is as important as ownership of the code.

Two specific items belong in this first stage. The platform is one patch release behind on Ruby, which we would bring current immediately. And the version of Rails it runs on moves from active support to security-only support in **October 2026** — inside this engagement's window. We would plan the framework upgrade as part of the first maintenance release rather than let a known, dated event arrive as a surprise later.

We would also ask to speak with the outgoing team during onboarding, and we would recommend ArtsAVL budget for a short paid overlap if they are willing. The documentation is at its most accurate the day its authors leave; their availability decays faster than the documents do.

### Assessment and discovery, before commitments

We would spend the first several weeks establishing facts rather than building, and we would produce written artifacts ArtsAVL keeps:

- **A platform assessment** against the actual code — architecture, maintainability, dependency health, performance and the practical limits of the current data model. This is where our technical recommendation moves from informed to verified.
- **Bounded discovery** with members, public users and every staff member who touches the administrative side. The RFP declines to hand us a feature list and asks for recommendations grounded in user needs; we agree with that, and the way to do it responsibly is a defined discovery with a defined output rather than an open-ended design phase.
- **An accessibility audit** with prioritized findings, so that conformance work is scoped against a measured baseline instead of a general commitment.
- **A tenancy architecture decision document** that settles how partner organizations would be separated, what each partner controls, and what it costs — reviewed and agreed before any of it is built.
- **Baseline measurement.** Member and profile counts, renewal rate, advertising revenue, event and opportunity volumes, traffic, and the staff hours currently spent on moderation and advertising trafficking. Without these, no improvement can be shown to have worked.
- **A phased roadmap** with sequencing, dependencies, effort and explicit decision points — including what is deferred and what deferring costs.

### A phased roadmap, and one honest observation about scope

The RFP describes six substantial goals, a six-month window, and a budget of $150,000–$250,000, and it invites recommendations where a firm believes a different scope or approach would fit better. We have estimated the full six-goal programme carefully and we do not believe it fits — not at a quality either of us would want to put ArtsAVL's name on. We would rather say that now than discover it in month four.

What we propose instead is to deliver a coherent, complete first phase inside the budget and timeline, and to hand ArtsAVL a costed, sequenced plan for the rest:

**Phase one — six months, inside budget.** Transition and continuity. Platform assessment and discovery. Accessibility audit and remediation of the highest-priority findings. Search and discoverability improvements, including structured data and sitemaps so that member profiles are found by the people looking for them. Staff reporting built on real platform data. The membership and advertising revenue workflows, including moving advertising purchase and the nonprofit package into self-service rather than email. And the tenancy architecture decision, fully specified and agreed.

**Phase two — defined and costed now, delivered next.** The multi-organization implementation and the first partner onboarding, plus expanded content distribution.

We are proposing to specify the licensing work in phase one and build it in phase two deliberately. It is the largest single piece of engineering in the RFP, it touches every record in the database, and it depends on decisions ArtsAVL has not yet had to make — chiefly whether partner organizations collect their own membership and advertising payments. Getting that answer before writing the code is worth considerably more than starting sooner.

If ArtsAVL would prefer all six goals inside the initial engagement, we will present that version with its cost stated honestly rather than trimmed to fit.

### Design and implementation

Design work proceeds from discovery findings and stays within the platform's existing conventions. We would work with ArtsAVL's established brand and, if there is a current design partner, alongside them rather than around them. Delivery runs in two-week increments, each ending with something ArtsAVL can see and a written record of what was decided and why. We would keep the platform's deliberate simplicity — server-rendered pages, no JavaScript build step — because it is the right choice for an organization without in-house engineers, and we would raise it with ArtsAVL rather than quietly change it if the design work ever pressed against that limit.

## 4. Licensing, scalability and content distribution

### How partner organizations would work

We would give each partner organization its own separately stored space inside one platform. Each partner gets its own web address, its own branding, its own members and listings, and its own local categories and regions — while ArtsAVL maintains a single system rather than one per partner. Each partner's records are held separately from every other partner's, which is the question a partner council's own board will ask, and it also means a partner who ever leaves can be handed their data cleanly.

Branding is handled so that partners can set their colours, logo and typography without anyone rebuilding the software, and we would validate colour choices for contrast so that a partner cannot inadvertently brand themselves into an accessibility problem. Categories and regions can differ per partner; we would recommend keeping enough shared structure that content can still be searched and syndicated across partners, and this is exactly the trade-off the tenancy decision document exists to settle with ArtsAVL.

We are deliberately not proposing per-partner payment collection in the first implementation. If partner organizations are to take membership and advertising money directly into their own bank accounts, that brings identity verification, payouts, refunds, disputes and tax reporting for each partner — a substantial piece of work that the RFP does not scope and that ArtsAVL may not need in year one. Our design accommodates it as a later addition without rework, and we would price it separately once ArtsAVL has decided.

### Scalability, honestly framed

The scaling question here is not traffic. It is the number of partner organizations and the operational load each one brings. A single shared application keeps hosting and maintenance costs flat as partners are added, which is what makes licensing worth doing at all; the costs that do grow per partner are metered services like mapping, storage and email. We would model that per-partner cost during discovery and give ArtsAVL a licence-pricing floor, so the annual fee is set with the economics known rather than estimated. It would be a poor outcome for ArtsAVL to sign partners at a price that becomes less profitable as the programme succeeds.

We would also be straightforward that each additional partner adds real operational work: a domain and certificate to configure, categories to set up, and staff at another organization who will have questions. Whether those questions come to ArtsAVL or to us is a decision worth making in the contract rather than discovering at the first support request.

### Content distribution

The platform already has an authenticated server-to-server interface for event syndication. We would leave its existing behaviour untouched — its consumers are not documented, and a change that breaks a partner nobody remembered would be a poor way to begin — and add alongside it the formats that news outlets, tourism organizations, libraries, municipalities and community calendars actually ingest: calendar feeds that subscribe directly, structured listing feeds, and a simple embeddable block that a partner site can drop in with one line and no technical work. We would also add structured markup to the public pages so that search engines and aggregators can read listings directly.

Each consumer would have its own access key with usage visible to ArtsAVL. That matters beyond housekeeping: knowing who consumes what is the prerequisite for ever charging for it.

## 5. Team, communication and the long-term relationship

### Who would work on this

`[NEEDS INPUT — named individuals, with short bios emphasising Rails depth and prior takeover experience.]`

The team we propose is a product lead, a project manager, a technical lead, two engineers, and a designer engaged through discovery and the design work. `[NEEDS INPUT — confirm names.]` No subcontractors. `[NEEDS INPUT — confirm.]`

On continuity, which the RFP asks about directly: we would name the product lead and technical lead in the contract and commit them for the maintenance term, not only the build. ArtsAVL is changing partners right now and knows exactly what it costs when the people who understand a system move on. The most credible answer we can give is a named commitment rather than a reassurance.

### What we would need from ArtsAVL

We would rather state this plainly than discover it later. The pace of this engagement will be set by ArtsAVL's availability more than by ours. Specifically, we would ask for an executive sponsor for decisions at each phase boundary, and **one named person as our day-to-day counterpart, at roughly eight to ten hours a week**, for prioritization, review and sign-off — plus a few hours from the staff who own moderation, advertising, reporting and programmes during discovery. If that counterpart's time is less than that, the consequence is honest and predictable: discovery lengthens, approvals queue, and dates move. We would flag it early rather than absorb it silently and let quality slip instead.

### How we would work together

Two-week increments, each with a demonstration and a written decision record that ArtsAVL keeps. A shared board for work in progress, with change requests handled in writing against the roadmap rather than absorbed informally. Prioritization is a joint conversation at each increment boundary, and ArtsAVL sets the order.

We put unusual weight on those written decision records, for a specific reason. ArtsAVL does not have an in-house technical counterpart to remember why a decision was made two years from now — so the written record has to serve that purpose. It is also what keeps ArtsAVL genuinely free to work with someone else in future, which the RFP's ownership terms make clear the organization intends to preserve. We think that is the right instinct and we would build for it rather than around it.

### Maintenance and support after the initial engagement

We propose an annual agreement covering one planned maintenance release each year — framework and dependency updates, security patches and performance work — plus as-needed support for defects and operational issues, with monitoring reviewed and documentation kept current.

`[NEEDS INPUT — response-time tiers and pricing.]` We would want ArtsAVL's support history before setting the price: roughly how many requests arrive monthly and how many hours the current arrangement consumes. Without that, any figure carries a contingency that neither side benefits from. If the history is not available, we would propose a provisional first-year rate with a true-up once we both have real data, which we think is more defensible than a padded flat fee.

The first annual release should include the Rails upgrade discussed above, since the support-status change falls inside this year.

## 6. Quality, security, accessibility and development practice

The platform arrives with more than 2,000 automated examples and 154 end-to-end journey tests, a guarded deployment process, dependency scanning and static analysis. We would treat all of that as an asset to maintain, not inherited overhead — every change ships with tests, and the existing suite runs as a release gate. Where we add multi-organization separation, we would add tests that specifically assert one partner's data can never be reached from another's context, because isolation is only as reliable as the code paths that respect it.

Documentation is updated as part of each phase rather than at the end, and stays in ArtsAVL's repository.

On security, we would preserve the existing posture — encrypted credentials, bot protection, rate limiting, transport security, dependency scanning — and keep card data out of the application entirely by retaining hosted payment pages, which is both safer and simpler. During onboarding we would inventory the credentials actually present in the application and rotate or remove anything not in active use, which is ordinary hygiene at a change of partner and a sensible first week's work.

On accessibility, we would start with an audit and a prioritized plan, then remediate the highest-impact findings within this engagement and give ArtsAVL a costed list of the remainder. We are proposing it this way because ArtsAVL publishes an accessibility commitment, and a general promise of conformance without a measured baseline is not a commitment either of us could verify. `[NEEDS INPUT — confirm target conformance level with ArtsAVL; the RFP asks for current standards without naming one.]` The per-partner branding work includes contrast validation so accessibility is preserved as partners are added.

## 7. Investment and value

`[NEEDS INPUT — all figures. Apply the blended rate to the hour envelopes in [[artsavl-estimation]]. Do not publish hour counts to the client; publish fees by phase.]`

| Item | Basis | Fee |
|---|---|---|
| Onboarding, transition and continuity | Fixed | `[NEEDS INPUT]` |
| Platform assessment, discovery, accessibility audit and tenancy architecture | Fixed | `[NEEDS INPUT]` |
| Phase one delivery — discoverability, reporting, revenue workflows, accessibility remediation | Fixed | `[NEEDS INPUT]` |
| Phase two — multi-organization implementation, first partner onboarding, expanded distribution | Estimated range, confirmed after the architecture decision | `[NEEDS INPUT]` |
| Annual maintenance and support | Annual, separate from the above | `[NEEDS INPUT]` |
| Per-partner payment collection, if required | Optional, priced separately | `[NEEDS INPUT]` |

**Assumptions behind the fees, stated so nothing is hidden:** ArtsAVL provides a named counterpart at eight to ten hours weekly · partner organizations do not collect their own payments in the initial implementation · one partner organization is onboarded, not several · accessibility remediation covers the prioritized tier identified by the audit, with the remainder costed separately · the already-completed modernization, testing, security and documentation work is not re-billed · travel `[NEEDS INPUT]`.

**Where we think the value is.** The most valuable thing in the first six months is probably not any single feature. It is that ArtsAVL ends the period with a platform that is genuinely maintained, an evidence-based plan it owns, and a decision about licensing made with real numbers rather than optimism. Alongside that, the improvements we have prioritized — being found in search, joining and renewing without friction, buying an advertisement without an email exchange, and staff seeing what is actually working — are the ones that act directly on revenue the platform already earns.

## 8. Schedule and capacity

`[NEEDS INPUT — confirm start availability and the capacity reserved for ArtsAVL.]`

Indicatively, against a six-month window from contract execution: onboarding and continuity in the first weeks; assessment, discovery, audits and the tenancy architecture decision through roughly week six, ending in a joint review of the roadmap and a confirmation of priorities; delivery in two-week increments thereafter, sequenced so that discoverability and revenue work do not depend on the licensing decisions and can proceed regardless of how those land.

We would ask ArtsAVL about fixed dates we should design around — board meetings, the State of the Arts Brunch, the 2027 Arts Guide production cycle, and membership renewal periods — since staff availability during those windows affects the pace more than anything on our side.

## 9. What we would want to know before finalising our technical recommendation

We have formed a clear view: keep and extend the existing platform. The evidence for it is the platform's own condition — current framework versions, substantial automated test coverage, a working deployment pipeline and maintained documentation. We considered rebuilding, including whether a different technology stack would make the multi-organization work easier, and concluded it would not: the difficult part of that work is the data and ownership model, which costs the same on any stack, and a rebuild would additionally mean paying again for a directory, calendar, opportunities board, membership, billing, advertising and administrative tooling that already exist and are tested.

To move that recommendation from well-founded to verified, we would want:

- **Access to the repository and its documentation under confidentiality**, ideally before final pricing. Our estimates for the multi-organization work rest on how the current ownership model is structured, and that is the one thing we cannot see from outside.
- **Whether partner organizations collect their own payments**, and how many partners ArtsAVL wants live within a year.
- **What must be identical across partners and what each partner controls** — branding, categories and regions, pricing, moderation rules, communications.
- **Current baselines** — members, renewal rate, advertising revenue, listing volumes and traffic.
- **Which email platform** sends the newsletters, and **which system** holds donor records, so we can recommend integration rather than duplication.
- **The accessibility conformance level** ArtsAVL expects, and any prior audit.
- **The support history** behind the maintenance agreement.
- **A conversation with the outgoing development team** during transition.

We would rather ask these now than price around them.
