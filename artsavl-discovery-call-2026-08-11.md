---
type: Note
related_to: "[[artsavl]]"
status: Active
---

# ArtsAVL — Discovery Call Notes (11 Aug 2026)

Present: Katie Cornell (Executive Director, ArtsAVL) · Camila Lopez (Growth Strategy Director) · Liam Shanahan (BD) · Matt Kortering (UX strategist). Source: `GMT20260811-151414_Recording.transcript.vtt`. Proposal committed for **15 Aug**.

Companion notes: [[artsavl-technical-assessment]] · [[artsavl-risk-register]] · [[artsavl-solution-architecture]] · [[artsavl-estimation]] · [[artsavl-proposal]] · [[artsavl-client-questions]] · [[artsavl-assessment-findings-vs-initial-read]]

---

## 1. Corrections to what we assumed — read this first

Four prior inferences are now settled by the client's own words. One of them reverses strategic advice I gave.

### Scale — my inference was wrong, and it matters

**Actual: "over 500 users and about 350 accounts."**

I had inferred *thousands* of profile records from autoincrement IDs observed on the live site (profile IDs running past 2214). That inference was wrong. The IDs are inflated by deleted records, unpublished drafts, and — the part I could not have guessed — a bulk of **free listings ArtsAVL created for every arts business in the county after Helene**, then used as outreach to convert to paid membership.

**Consequence: I retract the revenue reversal.** I had advised that membership was plausibly a six-figure annual line and that Goals 1 and 3 should therefore outrank Goal 4. At roughly 350 accounts × $75, membership is on the order of **$25–30K/year**, not six figures — and not all 350 accounts are paying, since the free listings sit in that count. Licensing genuinely is the growth thesis, exactly as the RFP framed it. My correction was itself the error.

### Timeline — much longer than the RFP says

**The RFP says six months. Katie says the real constraint is the fiscal year ending 30 June 2027, because the money is grant-funded.**

That is roughly ten to eleven months from award, not six. Liam's read on the call — "your timeline is wildly realistic… it scares me 0%" — is correct. This relaxes schedule risk substantially and changes the calendar in the estimate, though not the hours.

### Funding — confirmed, and available now

**Two grants: one restricted, one unrestricted.** Recovery funding, part of the strategic plan, use-it-or-lose-it. Asked whether they could start tomorrow: *"Yeah, we're good to go."* Closes open question A5.

### The outgoing developer — friendly, available, and already briefed

A **local freelancer who owns a bar in Asheville and is a musician**, doing development as a side business. He built the **Asheville Music Professionals** directory, which is how ArtsAVL found him — confirming the connection I inferred from Mindtonic Media's client list.

Critically: he is not leaving in acrimony. The platform has simply **"outgrown… one freelance guy doing it on a side hustle."** He **reviewed the RFP**, is *"all on board,"* and **bidders will get to talk to him**. This is the best possible answer to risk R3 and question A14, and it substantially de-risks handoff.

### Still unresolved after the call

**Little Green Light** was never mentioned — the donor-system question stands. **Linxup and Rastrac** were never mentioned — the two GPS-telematics entries in the RFP's integration list remain unexplained. And the **program manager** who runs the Creative Portal is described by role, not name; my earlier guess that this is Maria Buchanan (Communications Manager) is unsupported — the public staff page lists Selena Hilemon as Programs Manager. Treat the product-owner identity as still open.

---

## 2. Problems they named

Grouped by who feels the pain.

### Member-facing defects

- **Search is broken.** *"The search functionality has had some problems. People will add things and then cannot find them."* Members add a profile or listing and then cannot locate it. They have been working with the developer to make matching less dependent on exact entry. Katie called it a glitch that "needs to be addressed."
- **No analytics for members.** Ads carry tracking; **profiles, events and opportunities do not.** So ArtsAVL cannot tell a member how many people viewed their profile or listing — which is the evidence that would justify the $75 renewal. Katie: *"that would be very useful for us to be able to report back how much they're getting out of their membership."*
- **Filter set may be too broad.** Katie is actively debating it: *"do we have too many options on the filter? Do we need to narrow it down… is it good to have it robust?"* Genuinely undecided — a discovery question, not a defect.

### Staff and admin friction

- **No reporting.** *"Right now, we can't really run reports."* The only export is downloading the user list as a spreadsheet.
- **No in-tool messaging.** To decline or query a submission, staff must leave the system and send email from elsewhere. No record of what was sent.
- **Automated emails cannot be edited by staff.** Renewal reminders and profile-setup nudges exist, but *"our developer has to edit them for us."*
- **Page header content cannot be edited by staff.** Same dependency.
- **Moderation rests on one person.** A single program manager approves everything — profiles, events, opportunities — and also authors a large share of listings herself. Nothing goes public without staff approval.
- **Staff capacity is the stated ceiling on ambition.** Asked about membership tiers, Katie: *"We're not only open to it. Right now, we're limited by our own staff capacity."*

### Structural / business problems

- **The platform has outgrown its developer.** One part-time freelancer, and the licensing ambition makes that untenable.
- **Artists lack web presence — framed as a resilience problem.** Post-Helene, artists without websites or e-commerce were cut off when visitors could not reach Asheville. Katie frames getting them online as **emergency preparedness**, not marketing.
- **Duplicate event entry across community calendars.** *"Extremely time-consuming to add the same event to all the different community calendars."* This is the origin of the syndication idea.
- **Artists relying on Instagram as their only presence.** Partners like **Big Crafty** (two large vendor events a year) want *"something a little more professional"* than an Instagram page to link to when promoting artists.
- **Marketing support is the region's second-highest stated need**, from ArtsAVL's own post-storm community surveys.
- **Artopolis — the market alternative — is unaffordable and shallow.** Forward-facing only, no member back end, and priced for *"a million-dollar-plus organization."* ArtsAVL's positioning is deliberately the opposite: a sliding scale small councils can afford.

---

## 3. Existing features — confirmed on the call

| Area | What exists today |
|---|---|
| **Three public pages** | Profiles, Events, Opportunities. Public, no login required. |
| **Member dashboard** | Members edit their own profile, add event and opportunity listings, set their own **tags** |
| **Tag-driven personalization** | Member-set tags filter which grants, job listings and calls for artists surface in their dashboard. Applied mainly to events; **opportunities are not filtered as tightly** |
| **Profile content** | Description, contact, categories, **photo and video**, links out to e-commerce and ticketing |
| **"Open status" field** | Added post-Helene when it was unclear who was trading. Now vestigial — *"a functionality piece that we could probably take out at this point"* |
| **Moderation workflow** | Draft → submit for review → staff check against advertising/content policy → live. Declines are *"very rare"*, mostly content-policy violations |
| **Aggregation on profile** | Everything tied to an account surfaces on that account's profile page |
| **Membership — two levels** | Basic at **$75/yr**; **nonprofit sponsorship package** which bundles membership plus **two ad credits** |
| **Web advertising** | Shown on designated portal pages, and pushed to the main WordPress site **via iframes**. Placed primarily by arts businesses promoting an upcoming event; many redeemed through sponsorship credits. **Analytics exist here only** |
| **Automated email** | Renewal reminders, profile-setup nudges — not staff-editable |
| **User guides** | A member user guide **with video**, built before launch and kept current; plus an **admin user guide**. Katie is sending both — genuinely useful discovery input |
| **Growth mechanics** | Staff add major events proactively then notify the organization; weekly event roundups on social and email, with inclusion conditional on having listed the event |

---

## 4. What they want — planned, ranked by their own signals

### Katie's stated priority, when Liam forced the trade-off

Asked to choose between grand vision and getting the basics excellent, she chose quality without hesitation:

> *"Leaning more towards making sure that we have the best product… because we don't want to be offering something that's glitchy to other organizations. Make sure it functions really well."*

And immediately paired it with the commercial reason:

> *"We need to increase the earned revenue to continue to expand, to be able to invest more. Getting it to a point where we could at least license it or bring in additional cash flow… would allow us to continue to work on it and expand it."*

**Read: fix and harden first, then license. Licensing is the revenue engine that funds everything after.** That is a clear mandate and it settles the sequencing debate in our favour — polish before generalisation.

### Tier 1 — must be in the engagement

1. **Fix search.** Named as a defect, affects members directly, cheap relative to impact.
2. **Analytics on profiles, events and opportunities.** The single clearest member-value and renewal-justification feature. Katie raised it unprompted.
3. **Admin reporting.** Replace spreadsheet export with real reports.
4. **In-tool messaging to members.** Explicitly *"what we're really interested in"* — message during approval or decline, with history retained.
5. **Staff-editable automated emails and page headers.** Removes a standing dependency on a developer for content changes.
6. **Multi-organization licensing, with per-licensee approval rights.** Licensees must approve content in their own directory: *"we would want those that license it to be able to approve the things that show up in their directory."*

### Tier 2 — planned, wanted, not urgent

7. **Event syndication out to external community calendars.** Strong community rationale; Katie described it as a second layer rather than the core.
8. **Membership tiers** — base listing / customisable profile / performance visibility. *"Not only open to it"*, but gated by staff capacity, so tiers should not be built until staff can operate them.
9. **Narrowing or restructuring the filter set.** Undecided; belongs in discovery.
10. **Deeper social integration** from profiles.

### Tier 3 — named and explicitly parked

- **Community board for members to share updates with each other.** Real demand heard from members, but *"that might be out of our budget for right now."*
- **Removing the "open status" field.** Cleanup, no urgency.

### Explicit non-goals — decided, not open

- **No commerce and no ticketing.** Deliberate and well-reasoned: *"so many people use so many different things, we don't want to get into all of."* Profiles link out instead. Matt endorsed this strongly. Adjacent programs cover it — **Craft Your Commerce** at Mountain BizWorks provides coaching, and **Love Asheville from Afar** (with the local TDA) connects buyers to makers who ship.
- **No personalized grant recommendations.** Asked directly whether grants should be recommended or browsable, Katie chose **passive filtering** — members filter and find for themselves.

---

## 5. The licensing model — much more concrete than the RFP suggested

**Commercial shape**, modelled on Artopolis but repriced for affordability: a **one-time setup fee plus an annual maintenance fee**, on a **sliding scale**. Basis initially imagined as county population, revised on the call to **number of accounts** — Katie: *"it might not be on county population, it might be number of accounts."* Limits versus unlimited-with-reconciliation is unresolved: *"I don't think we've gotten that far yet."* Her instinct is to contact a licensee approaching their limit and offer an upgrade.

**Market size:** ArtsAVL works with **28 counties, roughly 26 arts councils**. Potentially ~25 licensees regionally, with no ceiling beyond that — state, then possibly out of state. But Katie is deliberately cautious: *"I don't want to do 30 different licensings. Start with a couple and make sure that we figured it out before we expand too much."*

**Demand evidence — this substantially answers risk B1.** Two surveys are already in the field: one to current members, one to peer arts councils and prospective licensees. Early returns: **4 licensing responses, 3 of the 4 interested**; 8 member responses after only a few days. Katie had not yet read the detail.

**Two named first licensees, both eager:**

- **CraftedWNC** — a regional craft initiative ArtsAVL is running, launching as a separate website, with ArtsAVL building the directory component. **ArtsAVL would sit on both sides — administrator and licensor.** That makes it close to an ideal first tenant: they can configure and test both roles without another organisation's politics or timeline in the way.
- **River Arts District Artists (RADA)** — roughly **700 artists**, on a directory platform that is *"super old and very glitchy to the point where it barely functions anymore."* Motivated by pain, and notably **twice ArtsAVL's own account count** — the first partner is larger than the host, which matters for how we size and sequence.

**Secondary benefit ArtsAVL cares about:** licensing produces a comprehensive regional database of artists, events and opportunities, which strengthens their **advocacy** work. Worth remembering — it is the unstated north star I flagged earlier, and it is real.

---

## 6. Useful context and signals

- **ArtsAVL is the designated arts recovery organisation for all of Western North Carolina**, contracted by the NC Arts Council after Helene. This is the mandate behind regional expansion, and it is why licensing is strategy rather than opportunism.
- **75-year-old organisation** — one of the oldest arts councils in the US, second oldest in NC. Katie: 8 years in role, 25 years in Asheville arts.
- **Three focus areas:** advocacy, grantmaking (5–6 programmes a year), and "connection" — promotion, which is where the portal sits.
- **Directory origin, 2022:** they evaluated Artopolis and rejected it specifically because it had no member back end.
- **Monthly service costs are expected and accepted.** Katie volunteered that Heroku runs about $100/month: *"yes, we expect that. We just need to kind of understand what they would be, so we can plan for it."* This meaningfully softens the §12 concern about disclosing subscription costs — she wants the number, not the absence of one.
- **Member technical proficiency is better than feared.** The user guide plus video *"helped a lot"*; some older artists need more hand-holding, but *"a lot of them are pretty tech-savvy."*
- **Membership geography** is currently Buncombe-limited, slowly widening to Western NC while staying regionally focused.
- **Nonprofit model pressure**, in Katie's words: *"the nonprofit model is going away, and so we have to be more forward-thinking."* That is the emotional driver behind the earned-revenue push.

## 7. Commitments we made on the call

- **Proposal delivered by 15 Aug.**
- **Two costed approaches** — "lowest hanging fruit" and "best world we can picture" — with effort ranges, so Katie can choose. This matches the phased shape already in [[artsavl-proposal]].
- **Grey areas called out explicitly** as assumptions or risks rather than silently priced. Liam: *"they're not recommending anything yet, because they don't know."* Consistent with the `[ASSUMPTION]` discipline in the assessment.
- **Tiered functionality levels** showing what each tier costs in workload — Liam offered, Katie agreed it would help.
- Katie to send the **member user guide and admin user guide**.

## 8. What to change in the proposal before Friday

1. **Retract the membership-scale argument.** §1 currently claims directory records "run into the thousands" and that membership is plausibly six figures. Both are wrong. Replace with the real figures — 500+ users, ~350 accounts — and drop the inference that Goals 1 and 3 outrank Goal 4 on revenue grounds.
2. **Reframe the timeline.** The binding date is **30 June 2027**, not six months. This makes the phased approach easier, not harder, and removes the schedule tension in §3 and §8.
3. **Name the two first licensees.** CraftedWNC and RADA turn the licensing section from hypothetical into concrete — and CraftedWNC being ArtsAVL-on-both-sides is the single best de-risking fact available for Goal 4.
4. **Soften the handoff risk language.** The outgoing developer is friendly, has read the RFP, and will speak to us. §3's paid-overlap recommendation should read as an easy yes rather than a warning.
5. **Add the four admin self-service items** — search fix, entity analytics, in-tool messaging, staff-editable email and header content — as named deliverables. These are Katie's own priorities and none of them are currently explicit in the proposal.
6. **Correct the accessibility and reporting assumptions.** [A-03] assumed reporting and communications largely exist and need enhancing. Katie says reporting effectively does not exist and messaging does not exist at all. **That Epic grows** — this is the assumption I flagged as the largest Goal 2 exposure, and it has resolved against us.
7. **Re-check the estimate.** Item 6 pushes hours up; the longer calendar and the friendly handoff pull risk down. Net effect needs recomputing before the fee table is final.
