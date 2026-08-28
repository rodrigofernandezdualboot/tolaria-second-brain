---
type: Project
---
# Infios

# Infios: The Opportunity

- Largest potential deal in Dual Boot history: estimated $12M to $15M (HubSpot placeholder set at $5M)
- In play since January; Alex has been working the AWS rep (Dan Glucci) since December 2024
- CEO meeting Monday: second meeting with CEO and CTO
  - CEO already signaled intent to move everything to AWS for maximum discount
- Two core workstreams:
  - ITS platform (Oracle Cloud migration): mission-critical customs/border compliance app
    - Outages cost customers avg. $50K/day; United Airlines is their largest customer
    - ~1,300 separate business applications need integration
  - Warehouse Management System: internal system of record, also on OCI
- Key pain points driving urgency:
  - Repeated outages causing customer churn and leverage on SaaS renewals
  - $1.2M in deals lost to competitors with more modern UX
  - Logic trapped in SQL Server stored procedures; no viable path forward on current stack
  - Developer turnover risk if they don’t modernize
  - Internal build attempt failed: engineers were pulled from existing workloads, not dedicated
- Microsoft was likely a competing option; now believed to have lost that battle
- All technical data sourced from Infios’ internal discovery (via Dan): do not reveal this to Infios directly

# Tech Stack and Capability Gaps

- Codebase is full Microsoft: [ASP.NET](http://ASP.NET) Web Forms, C#, C++
  - C++ presence suggests very low-level, performance-sensitive components
  - No deep technical context yet; Dan is a salesperson, not an engineer
- VM-to-EC2 migration expertise is a gap
  - Rodrigo to check with Matías on whether this capability exists internally
- Two partner options to fill gaps:
  - Digital Motion (Billy’s contact, Tom Miller): US-based, expensive, would white-label under Dual Boot
  - Platformer (existing partner, Danielle): AWS-funded, automates landing zones and account architecture, free to the customer
    - Matty is already trained on Platformer; preferred route to protect margins and lean into existing relationships
    - Blaine to set up a call with Danielle (Platformer) Monday to confirm fit for Infios
- Relatech also on the table:
  - Converts on-prem hardware from CapEx; can also provide staff augmentation (e.g. OCI expert)
  - Dan Glucci already connected Alex with Ryan Norton; partnership may already be signed
  - Treat as plan A alongside Platformer once formal requirements are known

# Funding and Deal Structure

- MAP funding: Dan running the assessment; estimated ~$1.2M in AWS funding available
- TD Synnex can handle MAP funding requests (takes ~10% cut) to reduce admin burden
- Deal likely structured in sprints, not one $15M RFP
  - Outcome-based milestones tied to cloud credit drops to self-fund continued phases
- AWS Box Program upside: 5 joint wins with a partner (e.g. Relatech, Platformer) unlocks $140K MDF per ISV
  - Goal: use this win to generate case studies and marketing blast to drive future pipeline

# Risks and Immediate Priorities

- Infios has had 3 to 4 failed partner engagements; their internal engineers are strong and will scrutinize the team
  - Must show up with genuine expertise, not overpromise and scramble later
- Sales org pattern of saying “we can do it” without confirming capacity is a known risk; must not repeat here
- Migration complexity: global client base (US, Europe, Africa, Asia) means regional availability and downtime windows need careful planning
- RFP expected imminently (possibly next week); architecture blueprints and formal assessments can’t start until actual discovery with Infios
- Everyone on the team needs to be aware now so there’s no scramble if things move fast after Monday’s CEO call

#

# RFP Bid Strategy Deck (Dualboot internal, Aug 2026)

Two separate deals, not one $15M RFP:
- ITS Platform Modernization: $400K, services over 4-6 months. Monolithic app, mixed Web Forms and MVC, 8-15 second page loads, zero test coverage. Business case shows 340% projected ROI over 3 years.
- WA (Warehouse Advantage) Migration: $5M, newly surfaced, high urgency. Triggered by a database outage that hit 1,500 machines and sent over 100 customers home for the day. Top-down mandate to migrate off current infrastructure.

Competitive field: shortlisted against Kayla and All Cloud. The Ironside Group is circling with a templated "Ascent" discovery offer. Dualboot's position: the partner that untangles messy production legacy systems generic SIs walk away from.

## Why Infios must act
- $50K/hour downtime cost when the app fails (trucks stuck at borders)
- 15% annual customer churn tied to performance issues
- $1.2M in deals lost to competitors with modern UX
- 40% projected developer turnover if they don't modernize
- Business logic trapped in 200+ stored procedures, some 30,000-40,000 lines, C/C++ linear logic with GO TO statements inside SQL
- Zero test coverage, 70% undocumented code
- Their internal team believes they can fix it alone — this is why the assessment matters

## Technical direction
- CTO mandate: "Get off SQL Server." Target architecture is AWS Aurora (confirmed by Dan Gallucci and the CTO)
- The older account strategy doc pointing to SQL Server 2025 is dead — every response/deck/conversation must align to Aurora
- Refactoring methodology (owned by Billy Boozer):
  1. Isolate stored procedures from the database, put under version control
  2. Chunk linear SQL logic into functional elements, map into an ORM layer (TypeScript/NestJS or .NET)
  3. Validate with an AI testing harness — custom agents stream data through both paths, verify new API endpoints match legacy stored procedure outputs 1:1 before shipping

## Capabilities split
Dualboot is not an infrastructure/VMware migration specialist. WA deal includes a large lift-and-shift of a legacy Oracle/VMware estate — formally subcontracting Digital Motion (Tom Miller and Ed) for DevOps/migration scope. Tom has run large infrastructure operations, including scaling Truth Social.
- Dualboot owns: database refactoring and stored procedure untangling, custom ORM translation layer, AI testing agent harnesses and 1:1 output validation, application modernization and new API surface
- Digital Motion owns: AWS landing zones and account architecture, VM-to-EC2 migration and containerization, orchestration infrastructure, DevOps rigor across the migration path

## Commercial strategy
- Phase 0: AWS-funded assessment. Dan Gallucci (AWS) offered to fund a Modernization Assessment through MAP as the tryout. Framed strictly as a low-risk audit producing a roadmap, never as an implementation commitment. Funding bounds still need confirmation from Dan.
- Milestone-based commercials tied to AWS cloud credit drops; services lock in against successful micro-migration stages
- ITS phasing already drafted: $800K foundation, $1.0M architecture, $600K AI innovation
- Core trust problem: the CTO has "only worked with bad partners" and believes the team can do this in-house. Phase 0 exists to prove otherwise — untangling the first batch of stored procedures during the audit is what wins the migration

## Questions to plant in the RFP evaluation criteria
Buyer-protective on their face, but disqualify templated competitors like Ironside in practice:
- "Which partner has shipped production agents that take actions in systems of record, not just RAG or chat, with measurable reliability and rollback patterns?"
- "Show me your evaluation harness for legacy stored procedures. How do you prevent regressions without exposing a live production database?"
- "How will you handle multi-tenant isolation, data sanitization, and secrets management in our messiest legacy customer environments?"

## Owners and deadlines before the RFP drops
- Draft the Phase 0 Modernization Audit proposal, scoped to untangle the first batch of stored procedures as proof — Sales/AE
- Build a one-page architecture blueprint of the AI evaluation harness and ORM transition layer — Solutions Architect (Rodrigo)
- Formalize the Digital Motion SOW with Tom Miller, covering VMware DevOps scope and subcontractor pricing — Operations/PM
- Meet Dan Gallucci to confirm MAP funding bounds and pre-wire AWS support for the bid — Alliance Manager

Goal for the week: walk into the RFP window with funding confirmed, the subcontract signed, and the audit proposal ready to hand over.
