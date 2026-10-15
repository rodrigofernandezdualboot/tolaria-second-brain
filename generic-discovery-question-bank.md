---
type: Note
belongs_to: "[[sales-engineering-role]]"
related_to:
  - "[[run-with-hal-client-question-set]]"
  - "[[artsavl-client-questions]]"
  - "[[hippoclinic-call-questions-2026-08-19]]"
  - "[[roofr]]"
  - "[[tag-open-questions]]"
  - "[[technical-intake-brief-pacific-seafood-as-400-invoice-flow-discovery-poc]]"
  - "[[park-road]]"
  - "[[wg-henschen-netsuite-integration-options]]"
status: Active
_favorite: true
_favorite_index: 2
_organized: true
---

# Generic Discovery Question Bank

Client-agnostic questions distilled from the question sets used on Run With Hal, ArtsAVL, HippoClinic, Roofr, TAG, Pacific Seafood, Park Road and WG Henschen. Pick from it; never run it top to bottom. A prospect who hears 40 generic questions concludes we did no reading.

## How to use this

**Read everything first.** Every question the prospect's own documents already answer costs credibility. Replace generic wording with their system, their numbers and their people before the call.

**Tier every question before the call:**

- **Tier 1 — Blocking.** The answer changes what we can propose or price. Ask the champion offline, before the main meeting if possible.
- **Tier 2 — In the room.** Phrased to show we understand the business. These earn trust as much as they collect data.
- **Tier 3 — Once engaged.** Real questions, wrong moment. Save for the technical follow-up or Phase 0.

**Frame sensitive questions as protection.** "Here's what we'd want to rule out for you," never "prove you own your product" or "why did your incident happen."

**Written vs. live.** In an RFP, written answers are often shared with every bidder. Ask questions that reveal our analysis, or that challenge the client's strategy, live. Never ask who else is bidding.

**Log what stays open.** Every unanswered question becomes an open question with an owner and a way to resolve it. Every inference becomes an `[ASSUMPTION]` with its impact if wrong.

---

## 1. Business problem and outcome

1. What problem are you solving, in your words, and what happens if you don't solve it this year?
2. What triggered this now? A contract expiry, an incident, a funding event, an acquisition, a vendor leaving?
3. What does success look like in 12 months, measured how?
4. What do you believe the system does today, and how confident are you that it matches what it actually does?
5. Where do you want this to be in three years? The answer decides how much we design for extension.
6. Is this a first phase of something larger? What comes after it?

## 2. Decision process and stakeholders

1. Who decides, and who signs? Is anyone whose approval is required not in this conversation?
2. Who is the economic buyer, and who is the technical owner day to day? A named person, not a shared mailbox.
3. Who is our primary counterpart, and how many hours a week can they give us?
4. Will a technical advisor or third party take part in evaluating proposals?
5. Who are the subject-matter experts we need for validation, and are they available?
6. Is a partner (AWS, a reseller, an outgoing vendor) involved in the deal, and what role do they expect to play?

## 3. Budget, timeline and commercial shape

1. Is there a budget range, and does spending it require further approval (board, investor, grant body)?
2. Where does the funding come from? Grants and credits often come with deadlines or usage restrictions.
3. Is the target date driven by something external: a contract end, a peak season, a regulatory date, an event?
4. When is your busy season? We need to know when your team is least available and when a cutover is safest.
5. Is there a hard deadline, and is there a later commercial backstop (for example, an old contract that keeps running) that gives rollback room?
6. Would you rather have the full scope delivered, or a prioritized subset done completely with the rest sequenced into a later phase?
7. Are you open to a short paid discovery or assessment before a fixed price?

## 4. Ownership, contracts and the incumbent vendor

Ask these early and privately. No architecture fixes an unfavourable contract.

1. Can we see the agreement with the current vendor, specifically IP assignment, data ownership, termination and transition-assistance clauses?
2. What is the notice period, and where is the relationship in its term?
3. Is there a data processing agreement? A processor must return data on instruction; an owner does not.
4. Has the request for data, code and account transfer been made in writing?
5. Which legal entity owns the developer accounts, cloud accounts, domains, app store listings and trademarks? Who receives the payouts or pays the bills?
6. Does anyone on your side have admin access to those accounts today, even read-only?
7. Who holds licences for third-party content, data or IP the product depends on?
8. Why is the current partner leaving, and are they bidding? (Live only, framed as transition risk, never as blame.)
9. Is the outgoing team available for a paid transition period?

## 5. Users, volumes and baselines

Nothing can be sized without these.

1. How many active users, accounts or tenants? How many paying?
2. What is the growth rate, and is usage seasonal?
3. What are the transaction volumes and peak rates (requests, writes, records per day)?
4. What revenue does the product generate, and through which channels?
5. Do you have analytics instrumentation, and whose account is it in?
6. How many support requests per month, and how many hours a year does support consume today?
7. Is there any channel that reaches all current users if the platform changes (email list, CRM, in-app)?

## 6. Current system and architecture

1. What is the stack, version by version? Which components are past or near end of support?
2. Where does the business logic live: application code, stored procedures, triggers, an integration layer, spreadsheets, people?
3. Is there documentation, an architecture diagram, or anyone who can draw one?
4. Is the codebase still being changed while we analyze or migrate it?
5. Which parts are custom versus off-the-shelf configuration?
6. What is the single part of the system everyone is afraid to touch?
7. Is there a staging or non-production environment, and how close is it to production?

## 7. Data

1. Where does the data live, which engine, which version, which hosting provider and plan?
2. How large is it, and how fast is it growing?
3. Which data is irreplaceable source data, and which is derived and could be regenerated?
4. After a record is created, how often is it read again? This sets storage tiering and archiving.
5. Are there tables without primary keys, generated columns, large blobs or spatial data?
6. Does the database support replication or change data capture, and do you have the privileges to enable it?
7. What is the retention obligation, and where does it come from (law, contract, habit)?
8. Who owns data quality, and what do you already know is wrong with the data?
9. If you can only recover part of the historical data, which part matters most to your users?

## 8. Integrations and third parties

1. List every system this one talks to, in both directions, and the mechanism for each (API, file drop, SFTP, webhook, queue, direct DB).
2. Which integrations run under your credentials, and which under a vendor's credentials?
3. Are any integrations on a deprecation path (SOAP, legacy auth, retired SDKs)?
4. Are there rate or concurrency limits shared with other integrations?
5. Who allowlists your outbound IPs, and who sends webhooks to you? Both break on a hosting change.
6. Do any third parties require contracts, approvals or certification before we can get credentials?
7. Who consumes your APIs or exports today, and what can they not get?

## 9. Identity, permissions and access control

1. How do users authenticate today: SSO, social login, email, a mix? What is the split?
2. How are permissions modelled: roles, row-level restrictions, per-tenant, per-field?
3. If you are multi-tenant, how is one customer's data isolated from another's?
4. Who has admin access, and is it standing or just-in-time?
5. Are there service accounts or shared credentials, and where are they stored?

## 10. Hosting, infrastructure and operations

1. Where is it hosted, which region, and how many accounts or environments?
2. Is infrastructure defined as code, or configured by hand?
3. What is the monthly cost, split by compute, storage and transfer?
4. What is the documented RPO and RTO, and has a restore ever been tested end to end?
5. Is there any copy of the data outside the primary account or provider?
6. How do you find out something is broken: your own alerting, a customer, or a vendor notice?
7. Can schema or infrastructure changes be gated for a short window during a migration?

## 11. Security and compliance

1. Which regulations apply: HIPAA, GDPR, CCPA, PCI, SOC 2, ITAR or export control, accessibility (WCAG level)?
2. Do you have signed agreements (BAA, DPA) with every vendor that touches regulated data, including AI providers?
3. Have you had a security incident? What was the root cause, and what changed afterwards? (Live, framed as "what we'd want to rule out for you.")
4. Is audit logging on for data access, not only for configuration changes?
5. When was the last penetration test, and are its findings closed?
6. Can external contractors push code to production?
7. Is there an asset inventory?
8. Does any data need to stay in a specific region or never leave your environment?

## 12. Engineering team, SDLC and knowledge transfer

1. Who maintains the system today: internal team, contractors, one person?
2. How many engineers work on it, and what would you want us to own versus advise on?
3. Is there automated testing, CI/CD, code review? How are releases done?
4. Who holds knowledge that isn't written down, and how long will they be available?
5. Can we get committed hours from that person for knowledge transfer?
6. What engineering conventions must we follow? Are there technologies you rule out?

## 13. Product and UX

1. Which features do users rely on most, and which do they complain about? (Check app store reviews and support tickets first.)
2. What must stay identical, and what are you open to changing?
3. Are there brand, design system or accessibility standards we must follow?
4. Is a design partner involved, or is UX in our scope?
5. How do you want to communicate changes to existing users, especially anything they lose?

## 14. AI initiatives

Use only when AI is in scope.

1. What decision or task should the AI make faster or cheaper, and how is it done today?
2. What does a wrong answer cost? Who reviews the output before it takes effect?
3. Which data would the model see, and is any of it regulated or confidential?
4. Must the assistant refuse when it doesn't know? How should it say so?
5. Which models and providers are allowed, and what are your data retention requirements for prompts?
6. How is AI spend tracked today, and do you need attribution by team or workflow?
7. Do users need live data, or is yesterday's data acceptable?

## 15. Scope, acceptance and success criteria

1. What is explicitly out of scope?
2. What does "done" mean: SME sign-off, an acceptance review, a metric hit, a go-live date?
3. Who signs off on each deliverable?
4. Which assumptions in our proposal would you push back on today?
5. If we find the scope was wrong after the first phase, how do you want to re-baseline?

## 16. What to ask for at the end of the call

Don't propose scope in a first call. Ask for the inputs that let us propose it:

1. Read-only access to code, schema or a sandbox, under NDA if needed.
2. An architecture diagram or account walkthrough, however rough.
3. A cost export for the last six months.
4. Existing requirements, process documents or swim lanes.
5. A follow-up with the named technical owner, without partners in the room.

---

## The five-minute version

If the call collapses to one exchange:

1. Who decides, and who owns the system day to day?
2. What triggered this now, and what is the date driven by?
3. Who owns the accounts, code and data today, and is there a contract that says so?
4. How big is it: users, data volume, monthly cost?
5. Can we get read access and an hour with the technical owner?
