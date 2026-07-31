---
type: Note
related_to: "[[artsavl]]"
status: Active
---

# ArtsAVL — What the Assessment Changed

Reconciles the Sol assessment of 2026-07-31 against the earlier team read in [[artsavl-overview]]. Written because several open items in that note are now answered, one assumption in the assessment turns out to be wrong, and two things point in opposite directions.

## The discrepancy I need resolved

**The assessment was run on the premise of a cold RFP with no client contact.** That was the stated input. But [[artsavl-overview]] records an **August 11 meeting with ArtsAVL**, an intro to the ArtsAVL contact coming from Liam, and Rodrigo and Peter leading prep.

If the Aug 11 meeting is real and still on, three things change:

1. **Risk B3 (possible non-live competitive process) drops substantially.** A meeting is the signal I said the Q&A response would have to substitute for. Pre-award investment can be uncapped with more confidence.
2. **Question routing changes.** Six questions in [[artsavl-client-questions]] were written for a channel where answers get circulated to every bidder. Live, they cost nothing competitively — and the three items I recommended holding back entirely become the most valuable things to raise on Aug 11, especially the challenge to the licensing thesis, because in conversation we can pair it with a cheap way to test it.
3. **Aug 11 sits four days before the Aug 15 deadline.** That is tight but workable: the meeting becomes the discovery conversation the written Q&A cannot be, and the proposal is written in the four days after it. Worth confirming the deadline hasn't moved.

**Action:** confirm whether the Aug 11 meeting happened, is scheduled, or fell through before the questions go out. It determines whether they are sent in writing or held.

## Open items in the earlier note that are now answered

| Earlier open item | Answer |
|---|---|
| "Current hosting unknown: AWS, Azure, GCP, or bare metal matters for scoping" | **Heroku**, per RFP §4. With Upstash Redis, DigitalOcean Spaces via ActiveStorage, GitLab for source and CI, and a guarded deploy pipeline. Heroku's per-app pricing is a real constraint on licensing economics — one app for all partners is cheap, one per partner is not. |
| "CRM name not specified in RFP: confirm before the meeting" | **Partially answered, and weaker than it first looked — see the evidence chain below.** ArtsAVL runs donations through a hosted form on `secure.lglforms.com`, almost certainly Little Green Light. Whether that is what Goal 2 means by "CRM integration" is *not* established. Ask, don't assume. |
| "Data cleanliness and volume are major variables" | Still unknown, and now framed as a specific ask: member and profile counts, events and opportunities per year, ad impressions per month, renewal rate. Also flagged: the platform's fourteen geographic regions are Asheville neighbourhoods, so the taxonomy will not transfer to a partner council. |
| Incumbent developer unnamed | **Mindtonic Media**, credited as code author in the meta tags and footer of every page on `connect.artsavl.org`. The RFP never names them. |
| "Ongoing maintenance (separate fee)" | Confirmed, and now dated: **Rails 8.1 leaves active support on 2026-10-10**, inside the engagement window. The 8.2 upgrade belongs in the first maintenance release rather than arriving as a surprise. |

### Evidence chain on the donor system — worked through because it was initially overstated

Recorded in full because this is the kind of claim that gets repeated in a client meeting, and the three steps have very different confidence levels.

1. **Observed, solid.** The Donate links on both `artsavl.org` and `connect.artsavl.org` point to `secure.lglforms.com/form_engine/s/mZJlS-ReDlMUi_d-VsziBA`. Fetched 2026-07-31: that URL serves ArtsAVL's own branded donation copy — "Keep AVL Creative!", Buncombe County residents, tax-deductible gift of $50/$100/$250/$500. So ArtsAVL has a live account on whatever service runs `lglforms.com`.
2. **Strong inference, undocumented.** That `lglforms.com` belongs to **Little Green Light**. LGL is the standard abbreviation, Little Green Light markets a product called "LGL Forms," and it is a donor management system built for small and mid-sized nonprofits — ArtsAVL would sit in its $45–60/month constituent tier. Little Green Light's own site does not state that this domain is theirs, so this is high confidence rather than certainty.
3. **Assumption — do not state as fact.** That Little Green Light is what RFP Goal 2 means by "CRM integration," and that donor records and platform member records are unlinked populations. **Nothing establishes either.** ArtsAVL may mean a different system entirely, or hold donors somewhere else as well. The "unlinked" reading rests only on donations leaving the platform for a hosted form, which is suggestive of separate systems but is not proof.

**Consequence for the question set:** the original wording — "Is Little Green Light the CRM referred to in Goal 2?" — was leading, because it hands the client our premise and invites a yes. Replaced with an open question: *what system holds donor records today, and what does Goal 2's "CRM integration" refer to?* That surfaces a system we have not seen if one exists, which the leading version would have concealed.

## Where the assessment disagrees with the earlier read

**On rebuilding.** The earlier note raises rebuilding on a new stack — .NET is mentioned — as potentially cheaper and as a valid recommendation to bring to the Aug 11 meeting with justification. **The assessment argues against it, on evidence rather than preference:**

- The platform runs **Ruby 4.0.5 (one patch behind current) and the Rails 8.1 series** — genuinely current, not nominally current.
- It carries **2,000+ RSpec examples and 154 end-to-end tests** covering the public site, member flows, payments and administration, plus a guarded deploy pipeline with code verification, security checks, database backup and migration.
- It has a maintained README, conventions guide, feature documentation, API reference, manual QA plans, integration references and a handoff inventory.
- **RFP §4 explicitly instructs that the proposal should not include the cost of rebuilding modernization, testing, security, documentation or administrative enhancements already complete.**

A rebuild recommendation would ask ArtsAVL to discard an asset they just paid to build and documented specifically to hand over — and §12's ownership terms show they are thinking about vendor lock-in, not looking for a fresh start. It would read as not having read the RFP.

**The multitenancy argument is the strongest part of the rebuild case and it still does not carry.** A framework that offers tenancy patterns out of the box does not remove the actual work, which is not the mechanism but the **data migration and the ownership-model redesign** — every record attributed to a tenant, every query scoped, taxonomies decomposed. That work is identical on any stack, and on a rebuild you also pay to reproduce the directory, events, opportunities, membership, billing, advertising, moderation and admin surfaces that already exist and are tested. The chosen approach — schema-per-tenant inside the existing Rails application — is in [[artsavl-solution-architecture]].

**Worth bringing to Aug 11 anyway**, but as the reasoning rather than the recommendation: showing that we seriously considered a rebuild and can say precisely why not is more persuasive to a non-technical buyer than never raising it.

## Where the earlier read was ahead of the assessment

**The accessibility audit idea is exactly right and I did not know it was already sourced.** [[artsavl-overview]] records that Liam's UX/accessibility contact could do a WCAG compliance audit in roughly two hours as a sales asset. That is precisely the mitigation for risk T5 — accessibility conformance is required to "current WCAG standards" with no level named and no baseline, which makes remediation unbounded and unpriceable. An audit before the proposal converts an open-ended commitment into a scoped one, and doing it unprompted demonstrates the diagnostic posture the RFP is asking for.

One caution: ArtsAVL publishes an accessibility commitment on its own site, so findings carry reputational weight for them. Present the audit as a contribution, not a critique of the outgoing partner.

**Looping in Echo early for UX** also maps onto a real gap — Goal 1's scope is undefined by design, and the 15% "approach to discovery, roadmap, UX and product strategy" criterion is where a bounded discovery proposal earns its points.

## What the assessment adds that the earlier read did not have

- **There is no technical evaluator at ArtsAVL and there cannot be one.** Katie Cornell is the economic buyer and the technical evaluator. Scoring is 20% understanding of the platform and business model, 20% experience, **5% cost**. The proposal wins on being repeatable to a board, not on architectural depth.
- **Katie personally owns the licensing market.** President of the Western Arts Agencies of NC, Immediate Past Board Chair of Arts North Carolina. Goal 4 is her strategy, sold into her own peer network — which is also why nobody internally will challenge it.
- **The licensing thesis may be untested.** No named partner, no published target, and peer councils as budget-constrained as ArtsAVL, which prices its own membership at $75/year. Validating it costs three conversations she is uniquely positioned to arrange.
- **Per-partner payment collection is unaddressed anywhere in the RFP** and is the largest unpriced item in Goal 4.
- **Two named integrations, Linkup and Rastrac, have no discernible purpose** in an arts directory and sit inside a codebase we would be restructuring.
- **The directory has no keyword search and its listings are absent from the initial page response**, with no sitemap declared — for a platform whose members pay $75/year to be findable.
- **Both the nonprofit package and web advertising are sold by email**, not self-service, which is where Goal 3's revenue friction actually lives.

## Standing note on evidence discipline

Public-inspection findings arrive at three confidence levels and the distinction has to survive into the client conversation: **observed** (in the page, fetchable, repeatable), **inferred** (a strong reading of observed evidence, but a reading), and **assumed** (a plausible connection with no evidence either way). The donor-system chain above is the worked example. The failure mode is not being wrong — it is collapsing three steps into one confident sentence in a meeting where nobody can check it, and where Katie Cornell would then repeat it to her board as ours.
