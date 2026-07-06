---
type: Project
_organized: true
---
# Sales Engineering Role

I'm starting this role, so what we need to build first is a shared knowledge base to be used in other proposals. This is what the [[lessons-learned-2]] is about.

## Sales Engineer as part of the Sales team

This [Notebook](https://notebooklm.google.com/notebook/4e2979d9-99d5-434b-9ad8-0a0bdc775dc8) contains all the documents that rule the sales process

### Full Funnel Process

Our sales process is strictly sequenced in HubSpot into five phases, and we cannot skip stages. Here is the path every lead must follow:

**Phase I: Lead Generation & Qualification:** Leads come in via marketing, events, or our outbound efforts. Our Sales Development Representatives \(SDRs\) act as the first human touchpoint, conducting discovery to document "BANT" \(Budget, Authority, Need, and Timing\). If qualified, the lead moves to Sales.

**Phase II: Sales Engagement & Deal Initiation:** A Business Development \(BD\) representative takes over and validates the BANT criteria. If the prospect is engaged and BANT holds up, the BD creates an official HubSpot Deal in the "Qualification" stage.

**Phase III: Internal Mobilization & Documentation:** The internal Deal Team is assembled. A dedicated Slack channel is created for the deal \(e.g., `[Client]-dealteam-sales`\), and a centralized NotebookLM repository is set up by the Commercial Office to host all transcripts and client documents.

**Phase IV: Proposal & Scoping:** This is where we define the project boundaries, estimate the effort, and draft the proposal.

**Phase V: Closing & Contracting:** Once an estimate is approved internally, the BD sends the proposal. Once we get a verbal agreement, we enter the Contracting stage with our Legal and Finance teams. Finally, upon signature, it is marked Closed Won and transitioned to the Delivery team.

**Where and When You \(The SE\) Come In**

As a Sales Engineer, you will primarily operate in **Phases III and IV**. You are our technical anchor.

**1. Joining the Deal Team \(Phase III\):** When a deal involves complex technical requirements, AI, or specific project builds, you will be added to the Deal Team's Slack channels. You will review the transcripts and documents in the internal NotebookLM to get up to speed on the client's business problem.

**2. Scoping, Estimating, and Prototypes \(Phase IV\):** When a BD determines that technical input is required to build a proposal, they will go to the `#commercial-office-inbound` Slack channel and submit a formal ticket. Operating under the rule *"If a request is not on the board, it doesn’t exist,"* this centralized ticketing system ensures your time is protected and prioritized

Here is exactly what you'll be doing when you receive a ticket:

**Technical Validation & Estimation:** You will review the request \(which is routed to a specialized `[Client]-dealteam-ops` Slack channel\) to translate business problems into technical boundaries. You will determine the project hours, the required roles, and identify "Nice-to-Haves" for the proposal. You have a **48-hour SLA** to complete scoping once all information is provided.

**Building Sales Prototypes:** Sometimes, we need a Proof of Concept \(POC\) to win the deal. If a BD requests a sales prototype, you will have a **5-business-day SLA** \(capped at 8 hours of total development work\) to deliver it.

**Owning the "Effort" Score:** We prioritize all internal tickets using a RICE framework \(Reach, Impact, Confidence, Effort\). While BDs own the deal's Impact, **you own the Effort dimension**. You will calculate the total person-days required to complete the ticket, which dynamically ranks your workload priority

## Basic team structure

| Role | FTE | Tasks |
| --- | --- | --- |
| PM | 0.25 | Product definition, team management, sprint planning and execution |
| TL | 0.25 | Technical guidance and execution |
| FE | 1 | Frontend development |
| BE | 1 | Backend development |
| QA | 0.5 - 1 | Quality assurance |
| DevOps | 0.25 | Infrastructure development |

This is a generic team-size reference for building estimates. The size varies from 2.5 to 4 FTE \(full-time employee\) depending on the scope of the project. Assuming a 4 FTE size, the team burns 160 hours per week.

## Do's

- **Compete on de-risking, not price.** Engineering hours are commoditizing — every firm scopes roughly the same number. Win by making the safe choice obvious: name the client's biggest fear and show a concrete method that retires it \(e.g. parallel-run + golden-case parity harness before any cutover\).
- **Anchor credibility in analogous prior work.** Map a near-identical past engagement to the prospect's problem point-by-point so expertise reads as evidence, not a claim.
- **Be precise about where the analogy differs.** Overclaiming kills credibility. Call out the gaps honestly \(TAG has no clean .NET upgrade path; TAG already made progress toward an isolated scoring module\).
- **Read everything before asking, then use the client's Q&A windows deliberately.** Show up understanding their system more deeply than competitors do.
- **Surface the landmines the client hasn't.** Name unknowns as unknowns and log them as open questions with an owner: scope ambiguity \(e.g. "Entry Level" undefined\), wrap-vs-rebuild feasibility, no client-side sandbox, golden test cases that don't exist yet.
- **Tag every inference** `[ASSUMPTION]`**; never fabricate a technical detail to look complete.** An honest gap beats a confidently wrong intake — it costs far more downstream.
- **Right-size the architecture to the client's operating capacity, not to what's impressive.** Modular monolith over microservices for a team with no ops function / a retiring developer. Say what should *not* be built yet.
- **Build tangible artifacts to raise authority.** Diagrams, sequence flows, and a working prototype / visual refresh move a deal more than prose. Show, don't tell.
- **Estimate honestly against the budget.** If a fixed fee is risky or insufficient, say so and quantify the drivers \(hours + unknowns\). Never under-bid to win.
- **Separate what the buyer** ***said*** **they need from what the system** ***actually*** **requires**, and frame Phase 1 as proof-of-partnership toward the multi-phase roadmap and the ongoing support the client will need.
- **Plan a re-baseline checkpoint for when key facts arrive** \(e.g. source code released post-award\) and identify + equip the technical champion who defends the choice internally.

## Don'ts

- **Don't compete on hours or rate alone** — it's a race to the bottom we don't control \(rates, costs, margin goals vary by firm\).
- **Don't take the happy-path description at face value.** "It's a simple integration" never is — probe the data model and auth.
- **Don't fabricate or overstate.** Never claim parity, feasibility, or a capability you can't yet prove.
- **Don't over-engineer or gold-plate** \(full multi-tenant SaaS, fine-grained microservices\) beyond what the client can operate and maintain.
- **Don't silently absorb out-of-scope work into the number.** Name it as an explicit value-add or scope item \(e.g. designer / UX refresh\), never hide it.
- **Don't share a prior client's name, materials, or sample outputs without confirming confidentiality first** — always clear what can actually be showcased.
- **Don't defer naming risks.** Unflagged dependencies and assumptions become contract problems later.
- **Don't promise stretch scope the budget and timeline can't hold** \(e.g. an AI phase\); be explicit about what's deferred and why.
- **Don't assume undocumented knowledge stays available** — with a retiring/solo developer, front-load structured knowledge transfer.
- **Don't present generic methodology.** Always apply it to the client's specific system and business.
