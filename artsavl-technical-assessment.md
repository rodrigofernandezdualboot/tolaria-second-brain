---
type: Note
related_to: "[[artsavl]]"
status: Active
_organized: true
---
# ArtsAVL — Technical Assessment (Phases 1–5)

Sol deal assessment run 2026-07-31 against the *2026 Creative Portal Expansion RFP*. Source of truth is the engagement folder (`engagements/artsavl-creative-portal/assessment.md`); this note is the recall version. Companion notes: [[artsavl-risk-register]], [[artsavl-solution-architecture]], [[artsavl-client-questions]], [[artsavl-assessment-findings-vs-initial-read]].

**Basis and limits.** RFP plus public inspection of `connect.artsavl.org` and `artsavl.org`. No source code, admin panel, member dashboard, hosting console, or member/revenue figures. Repository documentation goes to the selected firm at onboarding only.

## Client profile

- **Small nonprofit, designated local arts agency** for Buncombe County (Asheville Area Arts Council, founded 1952). Funded partly by the NC Arts Council and Buncombe County; a county commissioner sits on the board.
- **~5 staff, none technical.** Public staff page lists four: Katie Cornell (Executive Director), Rebecca Lynch (Development & Grants), Alexa Whitman (Policy & Research), Selena Hilemon (Programs). Maria Buchanan (Communications Manager) is named on the membership page but absent from the staff page. 16-member volunteer board with **no technologist**.
- **The platform was built and maintained by an outside agency: Mindtonic Media**, credited as code author in the meta tags and footer of every page on `connect.artsavl.org`. The RFP never names them. They are the outgoing partner.
- Design contractor: Mandy Torrence, credited across both properties.

### Buying group

- **Katie Cornell — economic buyer** ***and*** **technical evaluator.** Sole named RFP contact. Non-technical but systems-literate. Also President of the Western Arts Agencies of NC and Immediate Past Board Chair of Arts North Carolina — she personally convenes the peer councils the licensing goal would sell into. Goal 4 is her strategy.
- **Maria Buchanan** — probable day-to-day administrator and the closest thing to a functional product owner. Unconfirmed.
- Financial scrutiny on the board: Susan Harper (Treasurer), Alaina Nelson (Financial Advisor), April Brown (BofA SVP). Dodie Stephens (marketing professional) is the likely voice on UX. Brandy Bourne (Vice Chair, Library Director) is the only plausible technical-adjacent board evaluator — a hypothesis, not a fact.

### The central profile finding

**No internal technical champion exists, and one cannot be hired into existence.** A five-person nonprofit with no engineers cannot independently evaluate "enhance vs. partially redevelop vs. rebuild" — the exact judgment the RFP asks each bidder to defend. Katie will decide a technical question on non-technical criteria: clarity, confidence, coherence with her strategy, trust. The scoring confirms it — 20% understanding of the platform and business model, 20% relevant experience, **5% cost**.

Consequence: the recommendation must survive being repeated by a non-technical executive to a finance-heavy board, and it will go unchallenged — which means scope disagreements surface later as change orders rather than earlier as decisions.

## Problem diagnosis

**Stated problem vs. diagnosed problem** — the value is the gap:

| RFP says | Diagnosis |
| --- | --- |
| "Evaluate and recommend enhancement, partial redevelopment, or complete rebuild. We have not predetermined the approach." | Largely answered by ArtsAVL's own evidence — current Ruby/Rails, 2,000+ RSpec examples, 154 end-to-end tests, guarded deploy pipeline, maintained documentation, plus §5's instruction not to re-bill the modernization. The real decision is narrower: **which single subsystem must be reworked to carry licensing** — most likely the account model, which the RFP calls "the ownership and billing root." |
| Six goals, six months, $150–250K | Goals 1/2/3/5 are increments on a working product. **Goal 4 is a different product with a different operating model.** The budget funds one of those well. |
| "Long-term product development and technology partner" | What ArtsAVL structurally lacks is an **internal product owner**. "Partner" is the word available for a role a five-person nonprofit cannot hire. |
| "Improve UX based on user needs, not a predetermined feature list" | Goal 1's scope is **undefined by design** and no user research appears to exist. Largest commercial risk in the deal. |
| Goal 2's reporting/communications list | Half belongs outside the platform. ArtsAVL already runs two newsletters to 7,000+ subscribers at 60%+ open rate on an unidentified system. Integrate, don't build. |
| "Create a licensable, locally branded platform" | An **organizational transformation described as a feature.** Licensing makes ArtsAVL a software provider — partner onboarding, configuration support, a help desk, cross-tenant releases — carried by five non-technical staff. The RFP addresses the engineering half and none of the operating half. |

**North star:** recurring revenue from licensed partner organizations and the number of partners live — currently zero of both. Assumed 3–5 partners in year one; ArtsAVL has published no target.

**Recorded conflict.** The six goals are unranked, and Goal 1 (refine for Buncombe members) competes directly with Goal 4 (generalize for other councils) for one budget. A credible alternative north star exists that the RFP never states: **directory completeness as evidence for advocacy and public funding** — ArtsAVL is the *designated* agency and publishes creative-economy research, so a current directory is part of what justifies its designation and appropriations.

**Cost of inaction**, in ascending severity:

1. Foregone licensing revenue — probably tens of thousands, not hundreds. An organization pricing its own membership at $75 cannot charge peer councils enterprise rates.
2. **The handoff window closes.** Documentation is at peak accuracy the day its authors leave; their availability decays faster. Every month converts a prepared handoff into archaeology.
3. **The platform has no maintainer.** Eleven integration edges; membership renewals and ad purchases both run through Stripe. A webhook failure is revenue stopping on a day nobody chose, for an organization with nobody who can diagnose it.

## Current landscape

24 systems inventoried. **The maintainer field reads "nobody, imminently" for every application-layer system** — that is the phase's finding, not a gap in the research.

Disposition summary: core Rails app **Reuse** (extend, don't replace); PostgreSQL Reuse (tenancy lands here as schema change); Stripe Reuse but highest-risk under Goal 4; HAML/Bulma/Hotwire/importmap Reuse **with a decision point**; WordPress site Preserve with an interface contract; `ashevillearts.com` **Retire**.

### Four systems the RFP omits

- **Little Green Light** — donations on both properties route to `secure.lglforms.com`. Absent from §4's integration list. Almost certainly the CRM behind Goal 2's "CRM integration," with donors and members as unlinked populations.
- **The newsletter platform** — unidentified. Two newsletters, 7,000+ subscribers, 60%+ open rate, banner ads sold against them. A revenue channel with a larger audience than the membership base, on a system nobody named.
- `ashevillearts.com` — legacy domain still linked live from the platform footer's "Contact Us."
- WordPress 7.0.x with a Divi theme (inferred from theme query parameters).

### Two integrations with unknown purpose

**Linxup** and **Rastrac** appear in §4's integration list with no explanation and no obvious function in an arts directory. Rastrac's name matches a GPS/fleet-tracking product. Left as **Unknown** dispositions rather than guessed — either they carry real behaviour or they are dead dependencies, and both answers are useful.

### Observed on the live site

- On the directory, events and opportunities index pages, **filters and advertising are in the initial HTML response but the listings are not** — a loading indicator stands in, consistent with lazy-loaded Turbo Frames. No sitemap declared in `robots.txt`; `/sitemap.xml` returns nothing.
- The events page has a **keyword search field and date range; the directory has taxonomy filters and sort order but no search field.**
- Members pay **$75/year** to be findable, so this is a mission and revenue issue, not a technical nitpick.
- **The directory's fourteen regions are Asheville neighbourhoods and Buncombe towns** — DOWNTOWN Asheville, WEST: River Arts District, EAST: Black Mountain. Concrete proof the taxonomy will not transfer to a partner council.
- Ad creatives for events that had already occurred were still in rotation on 2026-07-31, consistent with flat monthly buys trafficked by hand. Both the nonprofit package and web ads are sold **by email**, not self-service.

### Revenue facts

Membership: one tier, **$75/year**. Nonprofit sponsorship package: **$250/year**, sold by emailing the Communications Manager. Web ads: **$100/month** for three placements, sold by emailing `hello@artsavl.org`. Newsletters: 7,000+ subscribers, 60%+ open rate. Print Arts Guide with advertising, 2027 edition reservations opening in October.

### Data and migration

No system is dispositioned Replace and the only Retire is a domain with no data — so there is no classical migration. Instead: **tenancy is a migration of everything.** Every record needs tenant attribution and every query needs tenant scoping, and the tenancy boundary lands directly on the existing ownership boundary. Largest technical work package implied by the RFP; the RFP does not mention it.

### The constraint that most shapes the solution

RFP §12: ArtsAVL retains ownership of data, branding, content, accounts, the existing codebase and **all enhancements**, and proposals must disclose any third-party software, licensing requirements, subscription costs or proprietary components affecting ArtsAVL's ownership, control or future use. In practice — no proprietary components, no encumbering licences, every paid dependency disclosed with its cost. **A client living through a partner transition is buying the ability to leave us as much as the ability to hire us.**

### The binding constraint

~5 non-technical staff, none dedicated, must supply discovery, design review, taxonomy and moderation decisions and acceptance testing across a six-month engagement touching every part of the product. This, not engineering capacity, sets the real pace.

## Verified framework currency

- **Ruby**: platform runs 4.0.5 (released 2026-05-20). Current is **4.0.6** (2026-07-14). One patch behind.
- **Rails**: platform runs the 8.1 series (released 2025-10-22, latest patch 8.1.3). **Rails 8.1 leaves active support on 2026-10-10** — inside the engagement window — moving to security-only until 2027-10-10.

The Rails 8.2 upgrade is therefore a dated, named event belonging in the first annual maintenance release, not a generic line item. And the platform is **genuinely current, not nominally current**, which substantiates the Reuse recommendation with evidence rather than assertion.
