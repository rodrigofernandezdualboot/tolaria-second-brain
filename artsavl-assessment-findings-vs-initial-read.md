---
type: Note
related_to: "[[artsavl]]"
status: Active
_width: wide
_organized: true
---

# ArtsAVL — What the Assessment Changed

Reconciles the Sol assessment of 2026-07-31 against the earlier team read in [[artsavl-overview]]. Also records three corrections to my own findings, because two of them would have been repeated in a client meeting.

## The August 11 meeting — what the source actually says

**The only source is [[artsavl-overview]].** Verbatim, it contains a section headed **"Prep Strategy for August 11 Meeting"** whose goal is to "arrive with recommendations and a position, not just questions" and to "show what can/can't be done within the $150K-$250K budget"; a line about looping in an Echo team member "into a strategy review before the call"; and a next step reading **"Make intro to ArtsAVL contact and confirm Rodrigo/Peter are leading (Liam)."**

**What that establishes:** an August 11 meeting exists, it concerns this RFP and this budget, a "call" exists distinct from an internal strategy review, and Liam owes an intro to an ArtsAVL contact.

**What it does not establish:** that the August 11 meeting is *with ArtsAVL*, or the year. I joined "August 11 Meeting" to "ArtsAVL contact" and reported it as "an August 11 meeting with ArtsAVL." It could equally be an internal Dualboot prep meeting on the 11th, with the client call separate and unscheduled.

**Why it matters, either way.** The assessment was run on the stated premise of a cold RFP with no client contact.

- **If the 11th is a client meeting:** risk B3 (possible non-live process) drops substantially, pre-award investment can be less cautious, and question routing changes — the six questions in [[artsavl-client-questions]] written for a channel that circulates answers to all bidders cost nothing live, and the three items recommended for holding back become the most valuable things to raise, especially the challenge to the licensing thesis, because in conversation it can be paired with a cheap way to test it. Aug 11 sits four days before the deadline: tight but workable, with the meeting doing the discovery work the written Q&A cannot.
- **If the 11th is internal:** B3 stands as scored, and the written questions should go out immediately — 15 days was already tight, and the RFP's question deadline coincides with its proposal deadline.

**Action: confirm which it is before the questions go out.** It is the single cheapest decision-unblocking fact available.

## Open items in the earlier note that are now answered

| Earlier open item | Answer |
|---|---|
| "Current hosting unknown: AWS, Azure, GCP, or bare metal matters for scoping" | **Heroku**, per RFP §4. With Upstash Redis, DigitalOcean Spaces via ActiveStorage (media served from `artsavl.nyc3.digitaloceanspaces.com`, so NYC3 region), GitLab for source and CI, and a guarded deploy pipeline. Heroku's per-app pricing is a real constraint on licensing economics — one app for all partners is cheap, one per partner is not. |
| "CRM name not specified in RFP: confirm before the meeting" | **Partially answered, and weaker than it first looked — see the evidence chain below.** Donations run through a hosted form on `secure.lglforms.com`, almost certainly Little Green Light. Whether that is what Goal 2 means by "CRM integration" is *not* established. Ask, don't assume. |
| "Data cleanliness and volume are major variables" | **First real scale signal:** profile IDs on the live directory run to at least **2214** (`/profiles/2113/…`, `/profiles/2214/…`). IDs are not counts — gaps, unpublished and lapsed records inflate them — but the order of magnitude is thousands of profile records, not hundreds. See the revision to the revenue read below. Also: the fourteen geographic regions are Asheville neighbourhoods, so the taxonomy will not transfer to a partner council. |
| Incumbent developer unnamed | **Mindtonic Media**, credited in a `meta-author` tag and a "Code by Mindtonic Media" footer credit on every page of `connect.artsavl.org`. A deliberate attribution, not a domain-name reading — the strongest of the inferences in this assessment. That they are *leaving*, and why, comes only from the RFP's phrase about preparing "for transition to a new development partner." Whether they are bidding is unknown. |
| "Ongoing maintenance (separate fee)" | Confirmed, and now dated: **Rails 8.1 leaves active support on 2026-10-10**, inside the engagement window. The 8.2 upgrade belongs in the first maintenance release rather than arriving as a surprise. Ruby is one patch behind (4.0.5 vs 4.0.6, released 2026-07-14). |

## Three corrections to my own findings

Recorded in full because each would have been repeated to a client, and Katie Cornell would then have repeated it to her board as ours.

### 1. The directory does have keyword search — I said it did not

Rendering `connect.artsavl.org/directory` in a real browser shows an `input[name=search]` with placeholder "Search". It is injected client-side, which is why it was absent from the fetched HTML. **The original finding was wrong.**

Root cause: an unrendered fetch of a Hotwire application returns the page shell. The listings genuinely are absent from the server response — a `turbo-frame#filters` with `src=/directory/filters` confirms deferred loading — but treating shell absence as *feature* absence is an inference, not an observation.

### 2. Profiles are indexable — I implied they might not be

`/profiles/{id}/{slug}` is **fully server-rendered**: complete body content, canonical URL, per-profile OpenGraph image, category taxonomy, published and modified timestamps. The concern that members paying $75/year might be invisible is largely retired.

What survives is narrower and still worth fixing: **no sitemap** is declared in `robots.txt`, `/sitemap.xml` returns nothing, and index-page listing links are absent from the server response — so the crawl *paths into* those indexable pages are weak. That is crawl discovery, not indexability, and a sitemap is a cheap high-certainty fix. Measure indexed coverage before proposing anything more.

### 3. The donor system — a three-step chain I collapsed into one sentence

1. **Observed, solid.** Donate links on both properties point to `secure.lglforms.com/form_engine/s/mZJlS-ReDlMUi_d-VsziBA`. Fetched 2026-07-31: it serves ArtsAVL's own branded copy — "Keep AVL Creative!", Buncombe County, tax-deductible gift of $50/$100/$250/$500. So ArtsAVL has a live account on whatever runs `lglforms.com`.
2. **Strong inference, undocumented.** That `lglforms.com` is **Little Green Light's**. LGL is the standard abbreviation, they market an "LGL Forms" product, and their donor management system targets small and mid-sized nonprofits — ArtsAVL would sit in the $45–60/month constituent tier. Little Green Light's own site does not state the domain is theirs. High confidence, not certainty.
3. **Assumption — do not state as fact.** That this is what Goal 2 means by "CRM integration," and that donor and member records are unlinked. **Nothing establishes either.** The unlinked reading rests only on donations leaving the platform for a hosted form.

**Consequence for the question set:** "Is Little Green Light the CRM referred to in Goal 2?" was leading — it hands ArtsAVL our premise and invites a yes, concealing any system we have not seen. Replaced with: *what system holds donor records today, and what does Goal 2's "CRM integration" refer to?*

## A revision to the revenue read

I originally wrote that membership revenue was modest at $75/year and that licensing was therefore the growth thesis. **The profile-ID evidence reverses that.** If the base is thousands of records and even half are active, membership is a six-figure annual line — plausibly comparable to this project's entire budget.

If so, optimizing conversion and renewal against an existing base may return more, sooner, and at far lower risk than building for partner councils that have not been named. **That strengthens Goals 1 and 3 against Goal 4** — and it sharpens the recorded conflict about which north star ArtsAVL is actually pursuing. Heavily caveated: IDs are not active-member counts. Needs the real figures before it is load-bearing in a proposal.

## Where the assessment disagrees with the earlier read

**On rebuilding.** The earlier note raises rebuilding on a new stack — .NET is mentioned — as potentially cheaper and worth bringing to the meeting with justification. **The assessment argues against it, on evidence rather than preference:**

- The platform runs **Ruby 4.0.5 (one patch behind) and the Rails 8.1 series** — genuinely current, not nominally current.
- **2,000+ RSpec examples and 154 end-to-end tests** covering the public site, member flows, payments and administration, plus a guarded pipeline with code verification, security checks, database backup and migration.
- Maintained README, conventions guide, feature documentation, API reference, manual QA plans, integration references and handoff inventory.
- **RFP §4 explicitly instructs that the proposal should not include the cost of rebuilding modernization, testing, security, documentation or administrative enhancements already complete.**

A rebuild would ask ArtsAVL to discard an asset they just paid to build and documented specifically to hand over — and §12's ownership terms show they are thinking about vendor lock-in, not a fresh start.

**The multitenancy argument is the strongest part of the rebuild case and it still does not carry.** A framework with tenancy patterns built in does not remove the actual work, which is not the mechanism but the **data migration and ownership-model redesign** — every record attributed to a tenant, every query scoped, taxonomies decomposed. That work is identical on any stack, and on a rebuild you also pay to reproduce the directory, events, opportunities, membership, billing, advertising, moderation and admin surfaces that already exist and are tested. The chosen approach — schema-per-tenant inside the existing Rails application — is in [[artsavl-solution-architecture]].

**Worth raising anyway**, as the reasoning rather than the recommendation: showing that a rebuild was seriously considered and saying precisely why not is more persuasive to a non-technical buyer than never mentioning it.

## Where the earlier read was ahead of the assessment

**The accessibility audit idea is exactly right and I did not know it was already sourced.** [[artsavl-overview]] records that Liam's UX/accessibility contact could do a WCAG audit in roughly two hours as a sales asset. That is precisely the mitigation for risk T5 — conformance is required to "current WCAG standards" with no level named and no baseline, making remediation unbounded and unpriceable. An audit before the proposal converts an open-ended commitment into a scoped one.

One caution: ArtsAVL publishes an accessibility commitment on its own site, so findings carry reputational weight. Present the audit as a contribution, not a critique of the outgoing partner.

**Looping in Echo early for UX** maps onto a real gap — Goal 1's scope is undefined by design, and the 15% discovery-and-product-strategy criterion is where a bounded discovery proposal earns its points.

## What the assessment adds that the earlier read did not have

- **There is no technical evaluator at ArtsAVL and there cannot be one.** Katie Cornell is the economic buyer and the technical evaluator. Scoring is 20% understanding of the platform and business model, 20% experience, **5% cost**. The proposal wins on being repeatable to a board, not on architectural depth.
- **Katie personally owns the licensing market.** President of the Western Arts Agencies of NC, Immediate Past Board Chair of Arts North Carolina. Goal 4 is her strategy, sold into her own peer network — which is also why nobody internally will challenge it.
- **The licensing thesis may be untested.** No named partner, no published target, peer councils as constrained as ArtsAVL. Validating it costs three conversations she is uniquely positioned to arrange.
- **Per-partner payment collection is unaddressed anywhere in the RFP** and is the largest unpriced item in Goal 4.
- **Two named integrations, Linkup and Rastrac, have no discernible purpose** in an arts directory and sit inside a codebase we would be restructuring.
- **A member's account email appears in the `article:author` meta tag on their public profile** (`meta-article:author: charly@venusrisingmedia.com`). Members do publish contact details deliberately, so this may be intentional — but raise it privately, not in a Q&A circulated to all bidders.
- **Both the nonprofit package and web advertising are sold by email**, not self-service, which is where Goal 3's revenue friction actually lives.

## Standing note on evidence discipline

Findings from public inspection arrive at three confidence levels, and the distinction has to survive into the client conversation: **observed** (in the page, fetchable, repeatable), **inferred** (a strong reading of observed evidence, but a reading), **assumed** (a plausible connection with no evidence either way).

Three claims in this assessment were stated at a higher level than their evidence supported — the CRM identification, the directory search, and the August 11 meeting. In each case the underlying observation was real and the compression happened at summary time. The failure mode is not being wrong. It is collapsing the steps into one confident sentence in a room where nobody can check it.

Practical rules taken from this: **render before concluding** anything about a Hotwire application's UI; **fetch one instance page** before generalizing about a whole content type; and **never let a summary state more than its source note does.**
