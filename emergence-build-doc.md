---
type: Note
related_to: "[[Emergence]]"
source: Emergence Build Doc.docx (Slack file F0BH3LA6HJB)
_organized: true
---

# Emergence Build Doc — Module Requirements

Full content of the client's requirements doc (client-authored, AI-generated per their email). This is the estimation source of record referenced in [[Emergence]]. Ten modules mapping to the four families in the client email.

## Module 01 — Company Brain (Intelligence & Knowledge Layer)

Operational knowledge lives across email, Slack, call transcripts and documents with no single place to ask a question and get a reliable, sourced answer. Every acquisition starts from zero — no structured record of what they know about each operating company's systems, contacts, entity structure and status. This module is both the answer to that and the shared data foundation the automation modules build on.

- Connectors and ingestion for unstructured sources: call transcripts, email (Google Workspace), Slack, and documents (Drive, deal-room artifacts), with deduplication and normalization
- User-controlled private data with granular sharing: a user controls their own private data and can grant specific tools or automations access to specific slices of it
- Source-level access controls enforced at query time: ingested content retrievable only by users who had access to the original source. No cross-boundary retrieval, no existence leakage
- Knowledge graph / ontology: structured entity layer over people, vendors, customers, deals, departments, operating companies, with entity resolution linking the same entity across sources into a unified timeline
- Must support automations and structured metrics on top. Two concrete test cases: extracting feature requests from sales calls; measuring user-feedback cycle time for product teams
- "Ask Emergence" Q&A: permissioned natural-language interface answering operational questions (e.g., an OpCo's EIN, registration state, integration status) with source attribution and a deep link to the original artifact

## Module 02 — AI-Pilling Tracker

They are pushing AI adoption across Emergence and the portcos and cannot see the truth of it: who uses what, for what, where the outliers are, and whether higher AI usage correlates with better team outcomes.

- Usage and adoption measurement, sliceable by operating company, department, individual, model (Opus vs. Sonnet vs. other), and tool (Claude Code vs. Cowork vs. API). Ingested from provider usage/billing exports, coding-tool admin APIs, and tool telemetry; identity mapped via SSO
- Current-state map: what AI is used for and where, plus outlier detection for follow-up
- Outcome-correlation engine (core of the module): maps AI usage against team outcome metrics — engineering throughput and quality (consumed from Module 03 where available, else vendor-instrumented as an interim feed) and business-team metrics where measurable — to surface whether and where higher usage drives better results

## Module 03 — Engineering Quality & Health

Engineering maturity varies widely across the portfolio with no continuous, comparable view of quality and deployment health. Recent audits surfaced representative gaps: no staging environment, single-reviewer PR bottlenecks, no automated security scanning. AI coding tools deliver far larger velocity gains on mature DevOps foundations, so this module gates where AI-engineering investment can go safely.

- Engineering health scorecard, instrumented from Git/CI across heterogeneous stacks; shareable benchmarking dashboard
- Architecture & pipeline audit tool: costed modernization plan per portco; detects foundation gaps that gate AI-agent deployment (staging, review bottlenecks, security scanning, test coverage). Recommends and optionally implements on request
- Shared reusable component library: auth, notifications, monitoring, vector search as opt-in configurable microservices
- Adoption tracking with avoided-rebuild-cost estimates in the shared dashboard

## Module 04 — Cloud Spend Intelligence & Consolidation

Cloud spend is fragmented across portcos with redundant services and no aggregate negotiating position. An affiliated infrastructure provider can absorb certain workloads at better unit economics than external vendors; that routing opportunity is invisible today.

- Billing ingestion and normalization across AWS, GCP, Azure per portco into a single allocated spend view
- Function-to-service map exposing redundant/suboptimal services for the same function, with recommended targets
- Volume-discount scenario modeling against migration cost and disruption
- Affiliated-provider routing analysis (identity/capability map under NDA at sprint) with side-by-side unit economics vs. the incumbent
- Consolidation/migration workflow surfaces: per-opportunity migration paths, CTO sign-off tracking, blocker visibility, adoption metrics

## Module 05 — Tech Stack Unification

They want a universal tech stack powering all portco products and internal builds, so architecture and codebases are recognizable across products — a developer fluent in one product understands another quickly. Blocker to declaring the stack blindly: they don't know how much of each portco's stack is immutable (compliance-bound or deeply embedded) versus unifiable.

- Reference stack and containerization standard: documented universal architecture and container standard (Docker/K8s) making CoE-built products plug-and-play across portcos and multi-cloud, no vendor lock-in; shared code and design repository with versioning and contribution model
- Portco stack inventory and classification tool: scans each portco's repositories and infrastructure (read-only, credentialed, no exfiltration), classifies components immutable vs. unifiable with confidence and rationale per component
- Compatibility scoring: per portco, how much of the stack could run on the reference layer and what the blockers are. Runnable against a new acquisition's stack during diligence/integration on a defined timeline

## Module 06 — Automation Platform

Technical and non-technical people across Emergence build AI-agent automations without one-off engineering, against a reliable design standard, with easy build/test/deploy and reliable integration with all their software tools.

- Agent automation runtime: compose, run, monitor AI-agent automations; usable by technical and non-technical builders
- Reliable automation design standard: opinionated pattern for predictable automations — error handling, retries, idempotency, approval gates, observability — so an automation built by one person is trustworthy to another
- Build / test / deploy loop: first-class testing (including dry-run against real data), controlled deploy path with versioning and rollback
- Integrations: reliable connectors to their software tools (CRM, email, Slack, calendar, docs, ticketing, HRIS, others named at sprint); clean path to consume the Module 01 data layer where present
- Full trace capture and a human-approval gate on any privileged or outbound action

## Module 07 — Competency Automation Layers

Fastest path to value: a data layer under an agent runtime (Claude Code / Cowork or equivalent) purpose-built per competency. For sales: a layer over CRM data, call data, and email enabling tools like automated call follow-ups, mining calls for feature requests, diagnosing closed-lost reasons. Pattern repeats for customer success, finance, operations, M&A.

- Sales competency data layer (first instance): unified, permissioned layer over CRM records, call transcripts, and email, structured for automation building — consuming Module 01 where present, standalone otherwise
- Reference automations: at least two production automations on the layer (e.g., automated post-call follow-ups; closed-lost / feature-request mining from sales calls)
- Build/test/deploy ergonomics so a non-specialist on the sales/ops team can build the next automation quickly
- Portability blueprint: documented pattern for standing up the next competency layer (customer success, finance, operations, M&A)

## Module 08 — AI Coworker for Non-Technical Staff

AI-pill the non-technical side on the model Ramp has published: a Cowork-style AI coworker, custom-built to act across all their tools, so non-technical staff accomplish real cross-tool work through natural language. Off-the-shelf Cowork is close but not wired into their specific tool estate or data layer.

- Non-technical AI coworker over the same agent runtime as Module 06 (or equivalent), able to read from and act across the tool estate on the user's behalf
- Deep tool integration with the systems non-technical staff live in (email, calendar, docs, Slack, CRM, finance/ops tools named at sprint)
- An adoption model, not just a tool: onboarding, starter library of common workflows, usage instrumentation feeding the AI-Pilling Tracker (Module 02)

## Module 09 — Automated Integration Management

Every acquisition triggers a 90-day integration playbook run largely by hand — gathering information about the acquired company, obtaining admin access to its tools, migrating it onto their standards (e.g., moving off other HR systems onto Rippling).

- Playbook execution engine running the 90-day playbook as a sequence of agentic and human steps, with status tracking and a per-deal record (ideally the Module 01 OpCo record)
- Information-gathering agents assembling the required picture of the acquired company (systems, contacts, entities, access map)
- Access-provisioning workflow obtaining and tracking admin access to the acquired company's tools

*Client will provide the current 90-day playbook at sprint.*

## Module 10 — Customer Health & Churn

Portcos lack a consistent, data-driven view of which customers are at risk and a reliable way to act before they churn. Churn/health-score model plus proactive, human-in-the-loop churn-outreach automation.

- Customer health / churn-risk model built on signals a portco actually has (product usage, support/complaint data, billing, engagement), with health score and ranked risk
- Proactive churn-outreach automation triggering intervention workflows on rising risk — drafting outreach and next-best-action, with human-approval gate before anything is sent
- Emission of health/churn metrics into shared benchmarking dashboards; consumes Module 01/07 data where present

## Cross-module dependency notes

- Module 01 is the shared data foundation: consumed by 02 (identity/usage context), 06 (data layer), 07 (standalone fallback if absent), 09 (OpCo record), 10 (data where present)
- Module 03 feeds Module 02's outcome-correlation engine and gates where AI-engineering investment can go safely
- Modules 06/07/08 share an agent runtime; 08's usage instrumentation feeds 02
- Items deferred "at sprint": Module 04 affiliated-provider identity (NDA), Module 06/08 tool lists, Module 09 90-day playbook
