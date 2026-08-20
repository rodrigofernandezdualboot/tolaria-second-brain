---
type: Project
status: Active
has:
  - "[[hippoclinic-call-questions-2026-08-19]]"
  - "[[hippoclinic-hipaa-self-assessment]]"
related_to: "[[rodrigo-fernandez]]"
_width: wide
---

# HippoClinic

> **Status:** pre-discovery, updated 19 Aug 2026 with the AWS meeting recap Blaine forwarded. First technical call today, 17:30 (UTC-3), 30 minutes, Zoom hosted by AWS.
> **Two sources, both secondhand.** Blaine's 12 Aug post in `#hippoclinic-dealteam-sales` (C0BPX8DSZPB) and AWS's own meeting recap, which Blaine forwarded on 19 Aug in C0BPTQ42S94. There is no architecture doc, no transcript, no RFP. Every technical statement here is client self-report relayed through AWS.

## The deal

AWS-originated, routed through Richard King (richgk@amazon.com). HippoClinic is a **medical / neuroscience SaaS platform** that acquires and processes brain signals to deliver diagnostic services. About **100 clinicians** access and analyze patient data on it, and the customer base is growing.

This is **not** an AI-spend play. Their workloads are classical signal and image processing — Fourier transforms, brain image segmentation, seizure onset localization — all running inside their own AWS environment. Bedrock (Claude) usage is ~$80/month of internal engineering coding assistance. It is an **infrastructure, security and cost-management** engagement.

A meaningful outcome of the AWS intro was simply that HippoClinic now has a named AWS account contact. They previously relied on general AWS technical support and found it ineffective. That absence is also why nobody has yet told them the things in the question set.

## Data and estate

| Fact | Value |
|---|---|
| Clinicians on platform | ~100 |
| Data per patient | 10–20 GB |
| Composition | raw brain signals, video, MRI |
| New data per month | ~2 TB |
| Storage | S3 |
| Accounts | **root billing account ending 6043, sub-account ending 6269** |
| Copies retained | **one** — redundancy declined on cost grounds |
| Bedrock spend | ~$80/month, internal coding assistance |

**Total stored footprint and monthly S3 spend are both unknown, and they size the whole deal.** 2 TB/month of growth says nothing about whether they are sitting on 20 TB or 500 TB.

## What they asked AWS for

Verbatim from the AWS recap:

- Confirmation of S3 data durability and reliability with a single-copy storage model
- HIPAA-compliant security architecture review and recommendations
- Guidance on backup and disaster recovery options
- Exploration of additional AWS credits to offset growing storage costs
- A dedicated point of contact for escalations and account support

Two of those five are ours. AWS has already recommended an architecture review; we are agreeing with their account team rather than competing with it.

The most useful line in the recap is that the team "is actively following internal quality and security procedures but is **unclear on what specific steps AWS can take** to further secure their platform." They do not know what to ask for. Naming it precisely is the value we add.

## Context on the three concerns

**1. Durability.** They keep a single copy of each patient's data in S3 to avoid the cost of redundancy. **Zua** raised the question of whether S3 can lose data under a correctly configured setup. Richard confirmed it should not, with proper configuration, and recommended an architecture review to validate what they have.

**2. Security.** A previous incident: leaked AWS SNS credentials were used to send thousands of unauthorized messages to international numbers, resolved only by shutting the service down. They are HIPAA-focused and want concrete, tailored recommendations rather than general guidance. Blaine's read is that this is where we add the most value.

**3. Cost.** ~$100K in credits received through Nvidia's Inception Program, with a similar application filed with AWS. Storage cost is rising rapidly as they onboard customers, and they are actively seeking further credits or funding.

## Our read — where the diagnosis differs from the ask

**The backup is not optional, and that collapses the cost objection.** The HIPAA Security Rule marks **data backup plan** and **disaster recovery plan** as *Required* implementation specifications — retrievable exact copies of ePHI, and procedures to restore lost data. Required means no risk-based opt-out, unlike Addressable specs where a documented alternative is acceptable. A single copy of patient data with no documented backup procedure is closer to an unmet obligation than a deferred expense. **Audit controls** are Required too, which is where the CloudTrail finding below lands. Mapping in [[hippoclinic-hipaa-self-assessment]]. This is the strongest lever in the deal: the thing they believe they cannot afford is the thing they are already required to have, and the archive-tier version costs a fraction of what they are imagining.

**They are asking the wrong durability question.** S3 Standard does not lose objects to hardware failure; it stores redundantly across at least three Availability Zones. What it does do is delete an object when something holding valid credentials tells it to. Given they have already had a credential compromise, their real exposure is **blast radius, not disk failure** — accidental or malicious deletion, a lifecycle rule pointed at the wrong prefix, a bucket policy change. The controls that answer that are versioning, Object Lock, MFA delete and an isolated backup account, not a second copy of everything. Reframing this is the single most useful thing we can do in the room.

**A two-account estate has no room for the separation this needs.** 6043 is the payer, 6269 is the workload account. So there is no separate log-archive account and, more importantly, **no separate backup account** — which means whatever can reach the data can also reach any copy of it. The concrete, tailored recommendation they asked AWS for and did not get is a third account whose only job is holding backups: Object Lock in compliance mode, and a role the workload account cannot assume. Cheap, specific to their incident, and it satisfies two Required specs at once.

**Durability and cost are not actually in tension.** Raw brain signal is write-once and almost certainly cold after the diagnosis is delivered. Archive tiering on cold data is roughly an order of magnitude cheaper than S3 Standard, so the saving plausibly funds the redundancy they believe they cannot afford. Whether that saving is worth $3K or $300K a year depends entirely on the unknown total footprint — ask before promising.

**The SNS incident has an unexamined second half.** A credential that could call SNS may have been able to call S3. Unless S3 **data events** were enabled in CloudTrail — they are off by default, only management events are logged — they cannot prove PHI was not read during the compromise window. Under the Breach Notification Rule an incident of unknown scope requires a documented four-factor risk assessment, and one of those factors is whether PHI was actually acquired or viewed. Without data events that factor is unanswerable. This appears in neither their list of asks nor the AWS recap.

**Credits are a bridge, not a fix.** The Nvidia Inception credits may not be spendable against an AWS bill at all — Inception credits generally apply to Nvidia's own programs. Worth confirming before anyone counts $100K against their infrastructure cost. And a credit that offsets a cost driven by per-patient data growth buys time without changing the trajectory; tiering changes the trajectory.

## Commercial shape

Our wedge is a **bounded, paid review scoped to durability posture and HIPAA security posture**, delivered against accounts 6043 and 6269, ending in a prioritised remediation plan. AWS has already recommended exactly this. Do not try to sell more than that on a 30-minute call.

## People

- **Zua** — HippoClinic. Raised the durability question in the AWS meeting. The only named person on their side so far, and the most likely technical champion. Confirm role and full name on the call.
- **Richard King** (richgk@amazon.com) — AWS, organizer of today's call.
- **engineering@hippoclinic.com** — the shared alias on the invite. Unknown who sits behind it.
- **Dualboot** — Blaine Wagner (deal owner), Matias Curbelo, Rodrigo Fernandez.
- **Peter Klayman** offered to deploy a BD; Blaine declined for now.

## Open items on our side

- Rodrigo has **no access** to the sales Drive folder shared by Camila Lopez (`drive.google.com/drive/folders/1inJAQLav6bg…`) — a search returns nothing. There is also a NotebookLM notebook. Ask Camila for access.
- Blaine posted the long brief on 12 Aug and separately said "intro meeting is on Tuesday with AWS and HippoClinic." The AWS recap is clearly a report of a meeting that already happened, so the sequence is still unclear — worth 30 seconds with Blaine before the call.
- **Unsettled and important:** is HippoClinic a Covered Entity or a Business Associate? It changes who is directly liable and which BAAs must exist in both directions. First question in the self-assessment.
- No BD assigned. No estimate, no scope, no budget signal.

## Related

- [[hippoclinic-call-questions-2026-08-19]] — question set for the 30-minute AWS sync, tiered by what fits in the room.
- [[hippoclinic-hipaa-self-assessment]] — the HIPAA checks HippoClinic should run on themselves, mapped to their estate. Holds the Required-spec argument that reframes the cost objection.
