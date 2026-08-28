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
