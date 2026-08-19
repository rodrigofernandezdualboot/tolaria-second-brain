---
type: Project
status: Active
has:
  - "[[hippoclinic-call-questions-2026-08-19]]"
related_to: "[[rodrigo-fernandez]]"
---

# HippoClinic

> **Status:** pre-discovery, created 19 Aug 2026. First technical call today, 17:30 (UTC-3), 30 minutes, Zoom hosted by AWS.
> **Everything below comes from one Slack message.** Blaine Wagner's 12 Aug post in `#hippoclinic-dealteam-sales` (C0BPX8DSZPB) is the only source. There is no architecture doc, no transcript, no RFP. Treat every technical statement here as unverified client self-report.

## The deal

AWS-originated, routed through Richard King (richgk@amazon.com). HippoClinic is a **medical / neuroscience SaaS platform** that acquires and processes brain signals to deliver diagnostic services. About **100 clinicians** on the platform, growing.

This is **not** an AI-spend play. Their workloads are classical signal and image processing — Fourier transforms, brain image segmentation, seizure onset localization. Bedrock usage is ~$80/month of internal engineering coding assistance. It is an **infrastructure, security and cost-management** engagement.

A meaningful outcome of the AWS intro was simply that HippoClinic now has a named AWS account contact. They previously relied on general AWS support and found it ineffective. That absence is also why nobody has yet told them the things in the question set.

## Data shape

| Fact | Value | Source |
|---|---|---|
| Clinicians on platform | ~100 | client |
| Data per patient | 10–20 GB | client |
| Composition | raw brain signals, video, MRI | client |
| New data per month | ~2 TB | client |
| Storage | S3, across **two linked accounts** | client |
| Copies retained | **one** — redundancy declined on cost grounds | client |

**Total stored footprint is unknown and it is the number that sizes this whole deal.** 2 TB/month of growth says nothing about whether they are sitting on 20 TB or 500 TB.

## The three stated asks

**1. Data durability.** They keep a single copy of each patient's data in S3 to avoid the cost of redundancy, and asked AWS whether S3 can lose data under a correctly configured setup. AWS said it should not, with proper configuration, and recommended an architecture review plus backup/DR guidance.

**2. Security.** They had a real incident: leaked AWS SNS credentials were used to blast thousands of international SMS messages from their account, and the only remedy they found was shutting the service down. They are HIPAA-focused and follow internal procedures, but want tailored recommendations rather than generic best practice. Blaine's read is that this is where we add the most value.

**3. Cost.** Storage cost is climbing as they onboard customers. They hold ~$100K in Nvidia Inception credits and have applied for AWS credits. Actively looking for further credits or funding.

## Our read — where the diagnosis differs from the ask

**They are asking the wrong durability question.** S3 Standard does not lose objects to hardware failure; it stores redundantly across at least three Availability Zones. What it does do is delete an object when something holding valid credentials tells it to. Given they have already had a credential compromise, their real exposure is **blast radius, not disk failure** — accidental or malicious deletion, a lifecycle rule pointed at the wrong prefix, a bucket policy change. The controls that answer that are versioning, Object Lock, MFA delete and an isolated backup account, not a second copy of everything. Reframing this is the single most useful thing we can do in the room.

**Durability and cost are not actually in tension.** Raw brain signal is write-once and almost certainly cold after the diagnosis is delivered. Archive tiering on cold data is roughly an order of magnitude cheaper than S3 Standard, so the saving plausibly funds the redundancy they believe they cannot afford. Whether that saving is worth $3K or $300K a year depends entirely on the unknown total footprint — ask before promising.

**The SNS incident has an unexamined second half.** A credential that could call SNS may have been able to call S3. Unless S3 **data events** were enabled in CloudTrail — they are off by default, only management events are logged — they cannot prove PHI was not read during the compromise window. Under the HIPAA Breach Notification Rule an incident of unknown scope requires a documented risk assessment, and "we had no logs" is the wrong answer to be holding. This is a serious point and it is not in their list of three asks.

**"Two linked accounts" is ambiguous and load-bearing.** An AWS Organization with service control policies and a separate PHI account is a different security posture from two accounts joined only for consolidated billing. If the application runtime and the PHI buckets live in the same account, the SNS incident could have been a data incident.

**The Nvidia Inception credits may not be spendable here.** Inception credits generally apply to Nvidia's own programs, not to an AWS bill. Worth confirming before anyone counts $100K against their infrastructure cost.

## Commercial shape

AWS has already recommended an architecture review plus backup/DR guidance. That is our wedge: a **bounded, paid review scoped to durability and HIPAA security posture**, delivered against their actual accounts, not a platform engagement. Do not try to sell the whole thing on a 30-minute call.

## People

- **Richard King** (richgk@amazon.com) — AWS, organizer of today's call.
- **engineering@hippoclinic.com** — a shared alias. We do not know who is behind it or what their role is. Finding the actual named technical owner is a first-call objective.
- **Dualboot** — Blaine Wagner (deal owner), Matias Curbelo, Rodrigo Fernandez.
- **Peter Klayman** offered to deploy a BD; Blaine declined for now.

## Open items on our side

- Rodrigo has **no access** to the sales Drive folder shared by Camila Lopez (`drive.google.com/drive/folders/1inJAQLav6bg…`) — a search returns nothing. There is also a NotebookLM notebook. Ask Camila for access; there may be material that changes the picture.
- Blaine posted the long brief on 12 Aug and separately said "intro meeting is on Tuesday with AWS and HippoClinic." The brief reads as a post-meeting report, so the sequence is unclear — confirm with Blaine whether the 18 Aug meeting happened and what came out of it before today's call.
- No BD assigned. No estimate, no scope, no budget signal.

## Related

- [[hippoclinic-call-questions-2026-08-19]] — question set for the 30-minute AWS sync, tiered by what fits in the room.
