---
type: Note
related_to: "[[artsavl]]"
status: Draft
_width: wide
---
# ArtsAVL Proposal

## 1. Our understanding of ArtsAVL and the Creative Portal

ArtsAVL has been Buncombe County's designated arts agency since 1952, and the Creative Portal is not a marketing site — it is a working business platform that carries part of the organization's unrestricted revenue. Membership at $75 a year, a $250 nonprofit sponsorship package, and $100-a-month web advertising are real income against a real cost base, and they sit alongside two newsletters reaching more than 7,000 subscribers at an open rate above 60%. That combination — public discovery on one side, a secure self-service member dashboard on the other, with staff review in between — is what distinguishes the platform from an ordinary directory, and it is why the RFP is right that this is a product rather than a website.

We also read the RFP as arriving at a specific moment. The platform has just been through a deliberate modernization and documentation effort in preparation for handing it to a new partner: current Ruby and Rails, more than 2,000 automated examples, 154 end-to-end journey tests, a guarded deploy pipeline, a maintained conventions guide and API reference. That is an unusually well-prepared handoff, and the RFP is explicit that none of it should be re-billed. Our reading is that ArtsAVL is not looking to start over. It is looking for someone to take custody responsibly and then build.

Three things we think matter more than they appear in the document.

**Licensing is the growth engine, and the numbers say so.** With roughly 350 accounts and 500-plus users, membership at $75 a year is a real but modest line — enough to matter, not enough to fund regional expansion on its own. That makes ArtsAVL's instinct correct: the path to a self-sustaining platform runs through licensing, not through squeezing more from the existing base. Two things follow. The free listings created for every arts business in the county after Helene are a known, sized conversion pool rather than a guess. And because the base is small, *proving* value to each member matters more than volume tactics — which is why we treat per-listing analytics as a revenue feature rather than a reporting nicety.

**Licensing is an organizational change, not a feature.** Making the platform available to other arts councils under their own branding turns ArtsAVL into a software provider: partner onboarding, per-partner configuration, questions arriving from other organizations' staff, and release management across more than one live instance. The engineering is the smaller half of that. We have priced and sequenced the engineering, and we flag the operating half plainly, because a small team taking on a support obligation is a real commitment that should be made deliberately.

**The current taxonomy is local by design.** The directory filters on fourteen geographic regions that are Asheville neighbourhoods and Buncombe County towns — Downtown, West Asheville, the River Arts District, Black Mountain, Fairview, Weaverville. A partner council in another county needs its own regions, while ArtsAVL will want enough shared structure that cross-partner search and syndication still make sense. That tension between local and shared is the central design question in the licensing work, and we would rather name it now than discover it during build.

**One thing we still could not determine from the outside.** Your description of the current technical environment lists two external services — Linxup and Rastrac — whose role in an arts directory we were unable to work out. We would resolve that during onboarding rather than assume anything about them, and if they turn out to be dormant, removing unused credentials is sensible first-week hygiene at a change of partner.

## 2. Relevant experience

The engagements below were chosen for their overlap with what ArtsAVL is asking for: maintaining and extending existing Ruby on Rails applications, directory and profile-driven products, revenue-generating platforms, multi-party systems, and relationships measured in years rather than months.

### Debtbook

#### Case Study

DebtBook, co-founded by Tyler Traudt and Erik Pelletier, is a platform that helps governments, educational institutions, and nonprofits manage financial data with precision. By replacing error-prone spreadsheets with a user-friendly application, DebtBook streamlines debt management, improves financial reporting, and enhances organizational impact. These sectors often face challenges with traditional spreadsheet-based methods, which lead to inefficiencies, data inaccuracies, and risks like fraud. DebtBook simplifies financial management, providing a solution that reduces errors, consolidates data, and minimizes risk.

#### The Solution

We collaborated with DebtBook from the conceptual stage to develop a robust application tailored to the financial management needs of public and nonprofit entities. We provided strategic guidance and validated the feasibility of the platform, ensuring it incorporated essential financial regulations and offered high accuracy in managing complex data. Our partnership extended beyond development, as we facilitated introductions to key vendors, media outlets, and investors, including securing a lead investor to support DebtBook's growth. The platform features an intuitive dashboard that simplifies data viewing, analysis, and reporting, while automating complex financial tasks to reduce reliance on error-prone spreadsheets. This innovative solution enhances decision-making, improves efficiency, and minimizes risks for organizations.

#### The Results

- Debtbook raised more than $23 million.
- Secured contracts with major educational institutions, local governments, and nonprofit organizations.
- We were able to efficiently build and launch the product in roughly 4 months.

### EntityKeeper

*Engagement from 2014, across multiple years and multiple roadmap phases · current status *`[NEEDS INPUT — confirm whether the relationship is still active]`

#### Case Study

EntityKeeper is a cloud-based platform that streamlines the management of legal entities for law firms, real estate investors, and business operators, with organizational chart visualization, compliance tracking and centralized records. It allows users to govern dozens — or hundreds — of entities with clarity.

By 2014, the platform's original **Ruby codebase had gone years without updates**. To meet the demands of a growing legal-tech market and ensure long-term scalability, EntityKeeper partnered with Dualboot to modernize the platform, accelerate development and drive product innovation. Revitalizing it required more than technical updates. It called for a partner who could stabilize and refactor an existing codebase, modernize the architecture and user experience, deliver new features aligned with client needs, scale the engineering team over time, **preserve institutional knowledge during transitions**, and collaborate on long-term planning.

#### The Solution

We began by stabilizing the inherited code, then transitioned into a full product development partner, collaborating with EntityKeeper's leadership over multiple years and across successive phases of the roadmap. We worked directly with the COO and product stakeholders, providing development, DevOps and product strategy.

The work included refactoring and updating the core architecture for reliability and scalability, and redesigning core modules: organization chart visualization for complex ownership mapping, compliance tracking with automated deadline reminders, entity profile management with centralized records for personnel, jurisdictions and properties, and role-based access controls for enterprise-grade permissions. Alongside it we implemented Agile practices — sprint planning, milestone tracking and improved DevOps workflows — scaled the team between two and six contributors as the roadmap required, **managed team leadership transitions without losing velocity**, and maintained communication through weekly meetings and shared Slack channels.

#### The Results

- Reduced the legacy bug backlog by **90% in the first six months**, laying the foundation for a stable release cycle.
- Launched **25+ new features** and enhancements.
- **Doubled release frequency** through improved planning, DevOps, and delivery process.
- Sustained a multi-year strategic partnership through changes on both sides — the closest parallel we have to what ArtsAVL is asking of a long-term partner.

### Boardroom Insiders

*Replatforming delivered over roughly three and a half months; new platform live at the end of 2018 · relationship ongoing*

#### Case Study

Boardroom Insiders, founded by Sharon Gillenwater, provides business intelligence on C-suite executives for enterprise sales, marketing and recruiting teams. It had grown into an online database of **more than 19,000 executive profiles**, maintained by a distributed team of editors and researchers.

The business was successful and the market growing, but the platform was built on a legacy PHP codebase that became more brittle with each addition. Neither Sharon nor President Lee Demby is a technical executive; they needed someone to own the platform so they could build a business on top of it. They had tried a contractor who managed the back end, then an outside agency, and described that process as painful — work shipped without proper testing, and they spent untold hours documenting defects. Meanwhile, their customers were getting larger and more demanding about user experience.

#### The Solution

Rather than add to a mountain of brittle code, we proposed replatforming the entire system — and we said so even though it was the bigger job, because it was what would let the business move faster later.

The original code was PHP on Rackspace and still functioning, so we built the replacement **in Ruby on Rails on AWS, in parallel**, while the existing platform continued serving customers as usual. When the new system was ready, we cut over. We led and managed the whole process, while giving Sharon and Lee direct access to our developers through a dedicated Slack channel. We deliberately preserved most of the existing public design so that the migration would shore up the foundation without disturbing a user experience customers were already happy with.

#### The Results

- At cutover there was **no uproar, no frustrated emails, no cancelled contracts — in fact no feedback at all** from public users, which for a migration of a live revenue platform is the outcome you want.
- The internal editorial experience, rebuilt rather than preserved, changed substantially: described by the client as "so much less cumbersome… more of a spa-like experience."
- Increased engagement from the editorial team and more focus on the higher-level analysis the product depends on.
- Follow-on work: an editorial dashboard for productivity metrics, performance tracking and staffing forecasts, then the company's first new product since founding — which Sharon credited to having replatformed first.

### PetScreening

*Design through public launch in under five months · public launch October 2017 · ongoing partnership, eight-plus years*

#### Case Study

PetScreening, founded by John Bradford — a long-time real estate investor, entrepreneur and North Carolina legislator — is a platform that streamlines pet screening for tenants and property managers. Traditional processes relied on self-reported forms with no verification, leaving property owners exposed to misrepresentation and liability, usually discovered too late to address.

John had built software before and knew what he needed: a development team, and someone to run point between that team and him.

#### The Solution

The engagement began with collaborative discovery sessions to map the business model, clarify requirements and align on go-to-market. Wireframes and mockups were created, reviewed and refined before engineering began.

Testing ran first inside John's own property management firm for two months, then as a closed beta with roughly ten additional firms — **generating revenue while still iterating**. Because the MVP reached market quickly, budget and flexibility remained to iterate on real user feedback, and we scaled the software team in step with adoption.

As adoption accelerated, onboarding demand began to outpace manual processes. The choice was to add staff or to automate. As John put it: *"Just throwing a person at it doesn't mean you'll get someone onboard… You have to make it automatic and instantaneous."* We built technology-driven onboarding workflows through system integrations and APIs — reducing friction, lowering cost of sale and letting the business scale without headcount.

#### The Results

- **21% month-over-month growth** in the period following launch, with customers arriving through inbound demand.
- Recognized as **Vendor of the Year** by a leading industry conference one year after public debut, and attracted interest from the largest multifamily management company in the US.
- Sustained uptime the client characterized as never having been down — freeing his attention from whether the platform works to what to build next.
- In 2025 alone: **$87.9 million** in reclaimed pet-related revenue for property managers and owners, **1.3 million administrative and legal hours saved** through the assistance-animal review process, **12,000 lost pets reunited** with families, and 5.1 million lost-pet alerts sent.

### PetScreening — infrastructure and scale on AWS

*Same engagement as above, infrastructure workstream · ongoing*

#### Case Study

As PetScreening moved from startup to established product, the platform had to absorb sustained growth in both traffic and data while keeping infrastructure cost under control. It now processes **up to 35,000 requests per minute against roughly 400GB of data**.

#### The Solution

We architected and operate the platform on AWS with a stack that closely mirrors the shape of ArtsAVL's own: **Amazon RDS for PostgreSQL** as the primary database with a read-only replica and automated snapshots, **ElastiCache (Redis)** for caching and as the store for **Sidekiq background jobs**, **ECS** for containerized application instances with autoscaling behind an Application Load Balancer, **S3** for photo and document uploads, **Lambda** for generating image thumbnails for social and third-party consumers, **CloudWatch** for monitoring, alerting and log aggregation, **CloudTrail** as an audit log, **SSM** as a parameter store, **MSK** for database synchronization, and **EC2 spot instances running CI/CD pipelines in GitLab**.

Cost was treated as a design concern rather than an afterthought: RDS Reserved Instances cut that bill by 20%, and Lambda handled image processing inside free-tier usage.

#### The Results

- Scaled from early-stage to **35k requests per minute and 400GB of data** without re-architecture.
- **20% reduction** in database hosting cost through reserved capacity.
- Read replicas and automated backups established for data integrity and availability.
- Platform in use by **over 200 property managers across 500,000 doors**, averaging four new property manager sign-ups per day, growing 22% month over month.

### MyWorkChoice

*Dates *`[NEEDS INPUT — not stated on our published case study]`* · platform live and operating at national scale*

#### Case Study

MyWorkChoice is a workforce management business specializing in temporary staffing and shift-based scheduling. Its platform is a **three-sided marketplace** connecting staffing agencies, client companies and shift-based workers, and today manages millions of shift assignments annually across multiple regions.

As the company expanded nationally, its legacy infrastructure struggled with growing transaction volumes and real-time communication demands. Manual scaling, fragmented monitoring and inconsistent cost management created bottlenecks: application slowdowns during peak shift-posting hours, manual server management and deployment complexity, idle compute capacity off-peak, and no centralized view of performance.

#### The Solution

We re-architected the platform into a cloud-native, containerized, cost-optimized AWS environment. **Amazon ECS on Fargate orchestrates Rails APIs, React front ends and Sidekiq background workers**, with automated scaling and zero-downtime rolling deployments. **RDS PostgreSQL with PostGIS** provides the transactional database and powers geolocation-based worker-to-shift matching and reliability scoring. **ElastiCache (Redis)** manages job queues and accelerates API responses across thousands of concurrent users. S3 stores profile photos, timecards and reports under lifecycle policies; Rekognition automates identity verification during onboarding; CloudWatch provides unified observability; Secrets Manager and KMS handle credentials and keys; VPC, Route 53 and load balancing provide secure multi-AZ networking. Infrastructure is defined in Terraform, with GitLab CI/CD.

#### The Results

- **99.9% uptime** across all production environments.
- **Two to three deployments per week with zero downtime.**
- **60% reduction in operational overhead** through automation and managed services.
- **35–45% monthly infrastructure cost reduction** via Fargate Spot, right-sizing and reserved capacity.
- **70% faster response times** during peak shift notification periods.
- Geolocation intelligence improved shift fill rates and reduced unfilled jobs.

### Why these five

Taken together they cover the areas your proposal requirements highlight. **Existing Rails applications:** Boardroom Insiders was rebuilt in Rails and is still ours; MyWorkChoice runs Rails APIs we modernized; EntityKeeper was an unmaintained Ruby codebase we inherited and stabilized. **Directory and profile products:** Boardroom Insiders maintains more than 19,000 curated profiles with an editorial workflow behind them; EntityKeeper manages entity profiles with centralized records and role-based permissions. **Revenue-generating platforms:** PetScreening and DebtBook were both built to earn, and PetScreening's shift from manual onboarding to automated self-service is the same move ArtsAVL needs for advertising and membership. **Multi-party products:** MyWorkChoice's three-sided marketplace separates agencies, employers and workers inside one system. **Long-term support:** EntityKeeper across multiple years and roadmap phases, PetScreening for eight-plus, Boardroom Insiders continuing past launch into new products.

The one we would point ArtsAVL to first is **EntityKeeper** — an unmaintained Ruby platform taken over from someone else, stabilized, modernized and carried for years, with continuity of understanding preserved across leadership changes on both sides. That is the situation ArtsAVL is in.

`[NEEDS INPUT — three referenceable clients with contact details, for comparable long-term product development or application-support engagements.]`

## 3. How we would approach the work

### Onboarding and continuity, first

Nothing else is safe until custody is established. Before any feature work, we verify our own access and ArtsAVL's ownership across every service the platform depends on — source control, hosting, payments, email, storage, caching, bot protection, monitoring, mapping and analytics — and we prove the critical paths still work by putting a live test transaction through payments and a test message through email delivery. We would also confirm that accounts and billing sit in ArtsAVL's name rather than a previous partner's, because ownership of the accounts is as important as ownership of the code.

Two specific items belong in this first stage. The platform is one patch release behind on Ruby, which we would bring current immediately. And the version of Rails it runs on moves from active support to security-only support in **October 2026** — inside this engagement's window. We would plan the framework upgrade as part of the first maintenance release rather than let a known, dated event arrive as a surprise later.

The most valuable thing available to us at the start is your current developer. You mentioned he has read the RFP and is happy to talk to us, which is genuinely the best position a handover can start from — most transitions do not come with a predecessor who wants it to go well. We would want a few structured sessions with him early, and if he is willing to be available on a light paid basis through the first weeks, we would recommend it. Documentation is at its most accurate while its author is still reachable.

### Assessment and discovery, before commitments

We would spend the first several weeks establishing facts rather than building, and we would produce written artifacts ArtsAVL keeps:

- **A platform assessment** against the actual code — architecture, maintainability, dependency health, performance and the practical limits of the current data model. This is where our technical recommendation moves from informed to verified.
- **Bounded discovery** with members, public users and every staff member who touches the administrative side — including the program manager who currently approves every submission, since her workflow is where most of the staff-time cost sits. The RFP declines to hand us a feature list and asks for recommendations grounded in user needs; we agree with that, and the way to do it responsibly is a defined discovery with a defined output rather than an open-ended design phase. Your member and admin user guides will shorten this materially.
- **An accessibility audit** with prioritized findings, so that conformance work is scoped against a measured baseline instead of a general commitment.
- **A tenancy architecture decision document** that settles how partner organizations would be separated, what each partner controls, and what it costs — reviewed and agreed before any of it is built.
- **Baseline measurement.** Renewal rate, advertising revenue, event and opportunity volumes, traffic, and the staff hours currently spent on moderation and advertising trafficking. Without these, no improvement can be shown to have worked.
- **A definition of "working well."** Because hardening the platform is the first priority, we would agree with you in writing what finished looks like — which defects are in scope, which are accepted, and what the exit test is. Quality mandates without a stated endpoint expand until they consume the schedule, and this one sits in front of the licensing work.
- **A phased roadmap** with sequencing, dependencies, effort, and explicit decision points — including what is deferred and what deferring costs.

### Scope and sequencing

All the goals in the RFP can be delivered within the stated budget. We have estimated them in detail, and we are not asking ArtsAVL to accept less than it asked for.

On sequencing, you have already articulated the answer better than we would have. You said you would rather have the best product than the biggest vision, because you do not want to offer something glitchy to other organizations — and then that you need the earned revenue licensing brings in order to keep investing. That is exactly the right order, and it is the plan we would propose: **harden first, license second, and let licensing revenue fund what comes after.**

So the first phase fixes and strengthens what exists, and specifies the licensing architecture in full — while the second builds it. There is a technical reason to keep that boundary as well as your commercial one. The multi-organization work is the largest single piece of engineering here; it touches every record in the database, and its shape depends on how the current ownership model is structured, which we cannot see until we have the code. Specifying before building is what stops us building it twice.

**What we would deliver in the initial engagement:**

- Transition, continuity and platform assessment.
- Bounded discovery with members, public users and staff.
- **Fixing search.** Members currently add a profile or listing and then cannot find it. This is first because it undermines the core promise of the directory.
- **Analytics on profiles, events and opportunities.** Today only advertising is tracked, so you cannot tell a member how many people viewed their listing. This is the evidence that justifies a renewal, which makes it a revenue feature rather than a reporting one.
- **Reporting for staff**, replacing spreadsheet export with reports you can actually run.
- **Messaging members inside the platform**, with a record of what was sent — so approving, querying or declining a submission no longer means leaving the system for separate email.
- **Content you can edit yourselves** — automated emails and page headers, without needing a developer.
- An accessibility audit with remediation of the highest-priority findings.
- Membership and advertising revenue workflows, including moving advertising purchase and the nonprofit package out of email and into self-service.
- Expanded content distribution, including pushing your event listings out to other community calendars.
- The tenancy architecture, specified, costed and agreed.

**What follows immediately:** the multi-organization implementation and the first partner onboarding, on the specification agreed in phase one.

On the first partner, one observation worth making. **CraftedWNC looks like the right place to start**, because ArtsAVL sits on both sides of it — administrator and licensor. Any remaining roughness is absorbed by you rather than by another organization, which protects exactly the concern you raised about not offering something glitchy. **River Arts District Artists is the better second**, with around 700 artists and a platform they say barely functions: real scale, real motivation, and a fair test of the model. Sequencing them that way lets licensing begin earlier without putting the quality bar at risk.

**If ArtsAVL would prefer the full programme, including the licensing build, inside the initial engagement**, we will present that version at its real cost. Two things we would want you to know if that is the preference: it consumes essentially the whole envelope, leaving little room for what discovery reliably turns up; and it commits us to building the tenancy model before we have read the code that determines its shape. We would take that on if asked. We would just rather you chose it knowingly.

### Design and implementation

Design work proceeds from discovery findings and stays within the platform's existing conventions. We would work with ArtsAVL's established brand and, if there is a current design partner, alongside them rather than around them. Delivery runs in two-week increments, each ending with something ArtsAVL can see and a written record of what was decided and why. We would keep the platform's deliberate simplicity — server-rendered pages, no JavaScript build step — because it is the right choice for an organization without in-house engineers, and we would raise it with ArtsAVL rather than quietly change it if the design work ever pressed against that limit.

## 4. Licensing, scalability and content distribution

### How partner organizations would work

We would give each partner organization its own separately stored space inside one platform. Each partner gets its own web address, its own branding, its own members and listings, and its own local categories and regions — while ArtsAVL maintains a single system rather than one per partner. Each partner's records are held separately from every other partner's, which is the question a partner council's own board will ask, and it also means a partner who ever leaves can be handed their data cleanly.

**Each licensee approves its own content.** You described a workflow where nothing goes public without staff review, and that a licensee would need the same control over what appears in their directory. That shapes the permission model directly: moderation rights are scoped to the partner, so a licensee reviews their own submissions without ever seeing another organization's queue.

Branding is handled so that partners can set their colours, logo and typography without anyone rebuilding the software, and we would validate colour choices for contrast so that a partner cannot inadvertently brand themselves into an accessibility problem. Categories and regions can differ per partner; we would recommend keeping enough shared structure that content can still be searched and syndicated across partners, and this is exactly the trade-off the tenancy decision document exists to settle with you.

We are deliberately not proposing per-partner payment collection in the first implementation. If partner organizations are to take membership and advertising money directly into their own bank accounts, that brings identity verification, payouts, refunds, disputes, and tax reporting for each partner — a substantial piece of work that the RFP does not scope and that ArtsAVL may not need in year one. Our design accommodates it as a later addition without rework, and we would price it separately once you have decided.

### Starting with two, not twenty-six

You have around 26 arts councils across the 28 counties you serve, and you were clear that you would rather start with a couple and get it right than sign thirty licensees. We agree, and the architecture is built for that: adding partners is configuration rather than construction, so starting small costs nothing in future capacity.

Your two interested organizations are also unusually well suited to being first. **CraftedWNC** is effectively a controlled pilot — you are both the licensor and the administrator, so you can configure and test both sides of the model without another organization's timeline in the way. **River Arts District Artists**, with roughly 700 artists on a platform that barely functions, is the case that proves the model works for someone else, at a scale larger than your own directory. The survey you have in the field will tell you who follows.

On the commercial model, the shape you described — a setup fee plus an annual maintenance fee on a sliding scale — is the right instinct, and moving the scale from county population to number of accounts is the better basis, because it tracks the cost a licensee actually generates. What is worth resolving before launch is whether licences carry a hard account limit or are unlimited with an annual reconciliation. We would model per-partner running cost during discovery and give you a licence-pricing floor, so the fee is set with the economics known. It would be a poor outcome to sign councils at a price that becomes less profitable as the programme succeeds.

### Scalability, honestly framed

The scaling question here is not traffic. It is the number of partner organizations and the operational load each one brings. A single shared application keeps hosting and maintenance costs flat as partners are added, which is what makes licensing worth doing at all; the costs that do grow per partner are metered services like mapping, storage, and email. You mentioned you already budget for monthly service costs and simply need to know what they will be — we would give you those numbers per partner rather than as a lump.

We would also be straightforward that each additional partner adds real operational work: a domain and certificate to configure, categories to set up, and staff at another organization who will have questions. Whether those questions come to ArtsAVL or to us is a decision worth making in the contract rather than discovering at the first support request.

### Content distribution

The platform already has an authenticated server-to-server interface for event syndication. We would leave its existing behaviour untouched — its consumers are not documented, and a change that breaks a partner nobody remembered would be a poor way to begin — and add alongside it the formats that news outlets, tourism organizations, libraries, municipalities and community calendars actually ingest: calendar feeds that subscribe directly, structured listing feeds, and a simple embeddable block that a partner site can drop in with one line and no technical work. We would also add structured markup to the public pages so that search engines and aggregators can read listings directly.

This is the piece that answers the complaint you hear from members — that adding the same event to every community calendar in the region is a waste of an afternoon. Enter it once with you, and it propagates. Each consuming calendar would have its own access key with usage visible to ArtsAVL, which matters beyond housekeeping: knowing who consumes what is the prerequisite for ever charging for it.

## 5. Team, communication and the long-term relationship

### The team we propose

We staff this engagement by role, with defined seniority and allocation, and we commit to that composition rather than to particular individuals:

| Role | Responsibility on this engagement |
| --- | --- |
| Project manager | Cadence, prioritization, change control, reporting, and the single point of contact for ArtsAVL day to day |
| Technical lead | Architecture decisions, the tenancy design and specification, code review, and the technical narrative for ArtsAVL's staff and board |
| Two engineers | Delivery across the platform — Rails, with production experience on this stack |
| Tester | Verification against the inherited automated suite, exploratory testing, and coordination of ArtsAVL's acceptance testing |
| Designer | Discovery research, user flows, and interface design through the experience work |

No subcontractors. Every person on the engagement is a Dualboot employee working under our own review and quality practices.

We do not put individual names into proposals, and we would rather explain why than list people we might later have to change. A named individual in a proposal is a commitment a firm of any size cannot honestly guarantee for a multi-year relationship — people take leave, change roles, and occasionally leave. We guarantee the structure, seniority, and rigorous practices of your team, ensuring consistent delivery that is never dependent on a single person. We staff by role and commit to the composition above, and we'd be glad to introduce the specific people during interviews.

### How continuity is actually maintained

Continuity comes from making sure understanding of the system never lives only inside the vendor. That is why the following are commitments rather than intentions:

- **The composition above is held for the engagement and for the maintenance term.** We do not reduce seniority after the contract is signed.
- **Changes to the technical lead or project manager role come with advance notice to ArtsAVL, a documented handover, and an overlap period at our cost, not ArtsAVL's.** Not more than one lead role changes at a time.
- **Every architectural and product decision is written down, in ArtsAVL's own repository, at the time it is made** — with the reasoning, the alternatives considered, and the trade-off accepted. This is the substantive continuity guarantee. It means a new person joining our team, or a different firm entirely, can reconstruct why the system is the way it is without needing to find whoever was in the room.
- **No single person holds exclusive knowledge of any part of the platform.** Code review across the team, shared access, and documentation kept current as part of each phase rather than at the end.
- **ArtsAVL's onboarding is documented as a runbook**, not as institutional memory — access, environments, deployment, and the operational checks that matter.

This is not a theoretical commitment. On EntityKeeper, we carried a multi-year engagement through leadership transitions on our own team without losing delivery velocity, and preserving institutional knowledge across those changes was an explicit objective of the engagement rather than a hope.

### What we would need from ArtsAVL

We would rather state this plainly than discover it later. The pace of this engagement will be set by ArtsAVL's availability more than by ours. Specifically, we would ask for an executive sponsor for decisions at each phase boundary, and **one named person as our day-to-day counterpart, at roughly eight to ten hours a week**, for prioritization, review and sign-off — plus a few hours from the program manager who runs the portal, and from the staff who own advertising and reporting, during discovery. If that counterpart's time is less than that, the consequence is honest and predictable: discovery lengthens, approvals queue, and dates move. We would flag it early rather than absorb it silently and let quality slip instead.

### How we would work together

Two-week increments, each with a demonstration and a written decision record that ArtsAVL keeps. A shared board for work in progress, with change requests handled in writing against the roadmap rather than absorbed informally. Prioritization is a joint conversation at each increment boundary, and ArtsAVL sets the order.

We put unusual weight on those written decision records, for a specific reason. ArtsAVL does not have an in-house technical counterpart to remember why a decision was made two years from now — so the written record has to serve that purpose. It is also what keeps ArtsAVL genuinely free to work with someone else in the future, which the RFP's ownership terms make clear the organization intends to preserve. We think that is the right instinct and we would build for it rather than around it.

### Maintenance and support after the initial engagement

We propose an annual agreement covering one planned maintenance release each year — framework and dependency updates, security patches and performance work — plus as-needed support for defects and operational issues, with monitoring reviewed and documentation kept current.

`[NEEDS INPUT — response-time tiers and pricing.]` We would want ArtsAVL's support history before setting the price: roughly how many requests arrive monthly and how many hours the current arrangement consumes. Without that, any figure carries a contingency that neither side benefits from. If the history is not available, we would propose a provisional first-year rate with a true-up once we both have real data, which we think is more defensible than a padded flat fee.

The first annual release should include the Rails upgrade discussed above, since the support-status change falls inside this year.

## 6. Quality, security, accessibility and development practice

The platform arrives with more than 2,000 automated examples and 154 end-to-end journey tests, a guarded deployment process, dependency scanning and static analysis. We would treat all of that as an asset to maintain, not inherited overhead — every change ships with tests, and the existing suite runs as a release gate. Where we add multi-organization separation, we would add tests that specifically assert one partner's data can never be reached from another's context, because isolation is only as reliable as the code paths that respect it.

Documentation is updated as part of each phase rather than at the end, and stays in ArtsAVL's repository.

On security, we would preserve the existing posture — encrypted credentials, bot protection, rate limiting, transport security, dependency scanning — and keep card data out of the application entirely by retaining hosted payment pages, which is both safer and simpler. During onboarding, we would inventory the credentials actually present in the application and rotate or remove anything not in active use, which is ordinary hygiene at a change of partner and a sensible first week's work.

On accessibility, we would start with an audit and a prioritized plan, then remediate the highest-impact findings within this engagement and give ArtsAVL a costed list of the remainder. We are proposing it this way because ArtsAVL publishes an accessibility commitment, and a general promise of conformance without a measured baseline is not a commitment either of us could verify. `[NEEDS INPUT — confirm target conformance level with ArtsAVL; the RFP asks for current standards without naming one.]` The per-partner branding work includes contrast validation so accessibility is preserved as partners are added.

## 7. Investment and value

> **INTERNAL — figures below are pre-call and now understated. Recompute before sending.** The reporting and communications Epic was estimated on the assumption that reporting, digest communications and activity tracking largely existed and needed extending (assumption A-03). Katie confirmed that reporting effectively does not exist beyond a spreadsheet export, in-tool messaging does not exist at all, and staff cannot edit automated emails or page header content. That is closer to new construction than enhancement and moves that line up materially. Pulling the other way: the calendar is longer than six months, and the outgoing developer is cooperative and available — both reduce risk rather than hours. Net direction is up. See [[artsavl-estimation]].

| Item | Basis | Fee USD |
| --- | --- | --- |
| Onboarding, transition and continuity | Fixed | $6,120 |
| Platform assessment, discovery, accessibility audit and tenancy architecture specification | Fixed | $33,575 |
| Initial delivery — search, member analytics, staff reporting, in-tool messaging, editable content, revenue workflows, accessibility remediation, content distribution | Fixed | *pending recompute* |
| Multi-organization implementation and first partner onboarding | Fixed, confirmed against the agreed architecture | $65,195 |
| Annual maintenance and support | Annual, separate from the above | `[NEEDS INPUT]` |
| Per-partner payment collection, if required | Optional, priced separately | `[NEEDS INPUT]` |

**Assumptions behind the fees, stated so nothing is hidden:** ArtsAVL provides a named counterpart at eight to ten hours weekly · partner organizations do not collect their own payments in the initial implementation · one partner organization is onboarded within the initial engagement, with a second following · accessibility remediation covers the prioritized tier identified by the audit, with the remainder costed separately · the already-completed modernization, testing, security and documentation work is not re-billed · new third-party subscription costs are disclosed with their monthly figure before we commit to them · travel `[NEEDS INPUT]`.

**Where we think the value is.** The most valuable outcome of the first phase is probably not any single feature. It is that ArtsAVL ends it with a platform that is genuinely maintained, a plan grounded in evidence rather than assumption, and a licensing model specified with real numbers attached. Alongside that, the improvements we have put first — search that finds what members added, analytics that show a member what their $75 bought, reporting you can run yourself, and messaging that stays inside the system — are the ones that make the product defensible enough to license, which is the condition you set.

## 8. Schedule and capacity

`[NEEDS INPUT — confirm start availability and the capacity reserved for ArtsAVL.]`

One thing worth settling early. The RFP asks for substantial completion within six months of contract execution, and we can work to that. You mentioned on our call that the real horizon is your fiscal year ending 30 June 2027, since the work is grant-funded. **We would rather build the schedule to whichever of those is the real constraint**, so it is worth confirming — and if the two grants have different performance periods, the earlier one is what actually governs. Grant deadlines usually require funds expended rather than work merely finished, so we would plan to complete ahead of the date with room for final invoicing and reporting.

Indicatively, from contract execution: onboarding and continuity in the first weeks; assessment, discovery, audits and the tenancy architecture specification through roughly week six, ending in a joint review of the roadmap and a confirmation of priorities; delivery in two-week increments thereafter, sequenced so that the hardening and revenue work do not depend on the licensing decisions and can proceed regardless of how those land.

We would ask about fixed dates we should design around — board meetings, the State of the Arts Brunch, the 2027 Arts Guide production cycle, and membership renewal periods — since staff availability during those windows affects the pace more than anything on our side.

## 9. What we would want to know before finalising our technical recommendation

We have formed a clear view: keep and extend the existing platform. The evidence for it is the platform's own condition — current framework versions, substantial automated test coverage, a working deployment pipeline and maintained documentation. We considered rebuilding, including whether a different technology stack would make the multi-organization work easier, and concluded it would not: the difficult part of that work is the data and ownership model, which costs the same on any stack, and a rebuild would additionally mean paying again for a directory, calendar, opportunities board, membership, billing, advertising and administrative tooling that already exist and are tested.

Our call answered a great deal — the funding position, the licensing appetite, your first two interested organizations, and the fact that your current developer is willing to help us take over well. What remains:

- **Access to the repository and its documentation under confidentiality**, ideally before final pricing. Our estimate for the multi-organization work rests on how the current ownership model is structured, and that is the one thing we cannot see from outside.
- **Which schedule governs** — the six months in the RFP, or your fiscal year to 30 June 2027 — and whether both grants run to the same date.
- **Whether partner organizations will eventually collect their own payments**, so we design the boundary correctly now even though we are not building it yet.
- **What must be identical across partners and what each licensee controls** — branding, categories and regions, membership pricing, moderation rules, communications.
- **Whether licences carry account limits** or run unlimited with annual reconciliation.
- **Baselines** — renewal rate, advertising revenue, listing volumes and traffic.
- **Which email platform** sends your newsletters, and **which system** holds donor records, so we recommend integration rather than duplication.
- **The accessibility conformance level** you expect, and any prior audit.
- **Support history** behind the maintenance agreement.
- **What Linxup and Rastrac do**, so we can support them properly or retire them cleanly.
