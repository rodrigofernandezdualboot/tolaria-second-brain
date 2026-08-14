---
type: Note
belongs_to: "[[wg-henschen]]"
related_to: "[[wg-henschen-preliminary-architecture]]"
status: Draft
_organized: true
---

# WG Henschen — Internal Message to David

Draft email, internal to Dualboot. Paste-ready. Detail behind it is in [[wg-henschen-preliminary-architecture]].

---

**Subject:** WG Henschen — two ways to build this, and what I need before we can price it

David,

I've done the technical research on the NetSuite assistant. Short version: it's buildable, there's no showstopper, and there's more real engineering in it than the conversation implied. Below is where I've landed, the one decision that drives the cost, and what I need from them before we put a number on anything.

**What's already settled**

Embedding the chat inside NetSuite is a solved problem — NetSuite has a supported way to run a React app inside its own interface, so we're not hacking anything. Signing users in works off their existing NetSuite login, no new accounts. And the "say *I don't have that information* instead of making something up" requirement is a native feature of the AWS service, not something we have to invent. Those three were the parts I expected to be awkward and none of them are.

**The actual choice: where the data lives when someone asks a question**

*Approach 1 — ask NetSuite directly, every time.* No copies of their data anywhere. Each question becomes a live query.

*Approach 2 — copy NetSuite into a reporting database on a schedule.* Questions get answered from our copy, not from NetSuite.

| | Ask NetSuite live | Copy to a reporting database |
|---|---|---|
| How current is the answer | Right now | As of the last sync — hourly or overnight |
| Good at "how many do we have" | Yes | No |
| Good at totals, trends, history | No, too slow | Yes |
| Their permissions | Come along automatically | We have to rebuild them |
| Build cost | Low | Meaningfully higher |
| Load on their NetSuite | Every question hits it | Sync only |

**My recommendation: build both, and let the system choose.**

The user asks one question in one box and never knows which path answered it. "How many of part 47-A do we have right now" goes live. "What's our total inventory value by category this quarter" goes to the reporting database.

It isn't fence-sitting — combining them is what keeps the expensive part small. Every question we can answer live is a question whose permissions we don't have to rebuild. So the hybrid isn't just better for the user, it's cheaper for us to get right.

**The one thing that will move the price the most**

We've decided the assistant must respect each person's existing NetSuite permissions. If a warehouse user can't see margin data in NetSuite today, they can't get it out of the assistant either. I don't think that's negotiable for a client of this profile, and I'd rather lead with it as a standard we hold than have it come up as a change request later.

Here's the catch. When we ask NetSuite directly, their permissions apply automatically and we get that for free. The moment we copy data into a reporting database, none of it survives the copy and we rebuild the whole access model by hand.

How much work that is depends entirely on how they've set it up, and we can't see that from outside:

- If access breaks into a handful of groups — warehouse, sales, finance — it's a modest piece of work.
- If it genuinely varies person by person, it's a substantially larger one.

That single fact is the biggest swing in the estimate. It's question 3 on the list below, and it's the one I most want to walk through live rather than over email.

**One assumption I want to correct before it reaches a proposal**

There is no ready-made connector between NetSuite and AWS's AI services. I checked all of them. The one AWS component that does speak to NetSuite has a hard cap of 100,000 records per pull and documented reliability problems with exactly the transaction and item data we need.

Getting data out of NetSuite reliably is custom engineering, and it's one of the larger line items. If anyone prices this as "wire up the connector," we'll be badly under. Worth saying out loud now.

**What I'd suggest commercially**

Don't quote a fixed number yet. We're missing three facts that each move the total, and I'd rather not absorb that risk or pad against it.

Propose a short paid discovery instead — roughly a week, sandbox access plus a couple of working sessions. It ends with a real architecture, a phased plan, and a defensible number. It also gets us in front of whoever owns their NetSuite, which is the relationship we need for delivery anyway.

Then phase one on inventory only. It's the domain where permissions are usually simplest, it's the use case they led with, and it gets something in front of their users fastest. Accounting and customer data follow once we've proven the pattern.

**Questions for WG Henschen**

Ordered by how much each one moves the estimate. I've written a fuller version with the reasoning spelled out — happy to send that instead if you'd rather forward something the client can read directly.

1. **Who administers NetSuite on their side** — internal, or an outside NetSuite partner? Everything else needs that person. Still unidentified from the last call and it's the critical path.
2. **Can we get a sandbox account** (not production), with API access? Half a day in a sandbox answers more than two more calls.
3. **How is data access restricted today** — by subsidiary, location, department, class, or by individual salesperson? A few broad groups, or person by person? Are there fields only some people can see, like cost or margin? *This is the big one.*
4. **What should it be able to answer** beyond inventory, and which NetSuite modules are they running? Best possible answer: ten to fifteen real questions they'd want to ask on day one, in their own words. That's how we scope it and how we'll prove it works.
5. **Three values from one screen** in NetSuite (Setup → Integration → Integration Governance): their service tier, SuiteCloud Plus licence count, and whether SuiteAnalytics Connect is licensed. Determines how much load the live path can carry.
6. **Is any part or drawing data export-controlled** — ITAR, EAR, CMMC? If yes it constrains where data can be processed and, potentially, who on our side can work on it. Worth checking internally first whether we already assessed this for the drawings project.
7. **Chat only, or do users need to export results** to CSV?
8. **Where should the assistant live** in NetSuite — its own menu item, a dashboard panel, or next to specific records?

**Two things I need on our side**

Maxim's AWS account access — I still can't see what's already set up in their Bedrock environment, and the demo is blocked on it.

And confirmation on the client's name. My notes say Henshon, the account says Henschen. Minor, but not something to get wrong in a proposal.

Rodrigo
