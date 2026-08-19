---
type: Note
belongs_to: "[[hippoclinic]]"
related_to: "[[hippoclinic]]"
_width: wide
---

# HippoClinic — Question Set for the 30-Minute AWS Sync (19 Aug 2026)

**Call:** Wed 19 Aug 2026, 17:30–18:00 UTC-3. Zoom, hosted by Richard King (AWS).
**In the room:** Richard King (AWS), engineering@hippoclinic.com, Blaine Wagner, Matias Curbelo, Rodrigo Fernandez.
**Prepared from:** Blaine's 12 Aug brief in `#hippoclinic-dealteam-sales`. No architecture doc, no transcript, no prior technical contact.

---

## How to use this

Thirty minutes with AWS in the room is not a discovery call. Realistically you get **six or seven real questions** and one idea worth remembering. Everything else is introductions and scheduling.

So the goal is narrow:

1. Land **one reframe** that shows we understand their problem better than they stated it.
2. Get the **four facts** that decide whether this is a small review or a real engagement.
3. Leave with a **named human** and a next meeting.

Do not run the security incident as an interrogation. They already shut a service down over it; they know it went badly. Frame every security question as "here's what we'd want to rule out for you."

---

## The one thing to say before you ask anything

> "Before we get into it — the way the durability question is framed may be doing you a disservice. S3 doesn't lose objects; it stores every one across at least three Availability Zones. What it does do is delete an object the moment something with valid credentials asks it to. Given you've already had a credential get out, we'd look at this as a blast-radius problem rather than a hardware problem — and the fixes are different. Versioning, Object Lock, an isolated backup account. Not a second copy of everything."

Then immediately, so it doesn't land as criticism:

> "And the reason that matters commercially is that you told AWS you're keeping one copy because redundancy is expensive. Our expectation is that most of two terabytes a month is write-once brain signal that nobody reads after the diagnosis goes out — and archive tiering on that is roughly an order of magnitude cheaper than what you're paying now. There's a good chance the saving pays for the protection outright. We'd want to check your numbers before promising that, but we don't think you have the trade-off you think you have."

That is the whole pitch. It costs ninety seconds, it is technically correct, and nobody else has said it to them.

---

# TIER 1 — Ask these today

Six questions. In this order.

### 1. Who's on the call, really?

> "Before we dive in — who owns the AWS estate day to day on your side? Is that you, or is there a platform person we should have in the room next time?"

*`engineering@` is a shared alias. We do not know if we are talking to a CTO, a lead engineer, or a rotation. Everything about how to run the next 25 minutes depends on the answer, and the technical champion has to be a person, not a mailbox.*

### 2. When you say a single copy — what exactly is that?

> "Is that S3 Standard with versioning switched off, or is it One Zone-IA?"

*This is the highest-information question in the set and it takes ten seconds to answer. **S3 Standard with versioning off** means the data is genuinely durable against hardware failure and the exposure is deletion — a governance fix, cheap. **One Zone-IA** means they have taken a real durability reduction: a single Availability Zone, no resilience to the loss of that AZ. That is a different conversation and a more urgent one. If the answer is "I'd have to check," that itself tells you the storage class was never a deliberate decision.*

Follow-on if there is room: is versioning on anywhere, and is there any Object Lock or MFA delete on the PHI buckets?

### 3. What are the two accounts, structurally?

> "You mentioned two linked accounts — is that an AWS Organization with SCPs, or two accounts joined for consolidated billing? And does the patient data sit in the same account as the application runtime?"

*The security answer for the whole estate turns on this. Two accounts under an Organization with a separate PHI account is a defensible design. Two accounts linked for billing is one blast radius wearing two hats — and it means the SNS compromise had reach into the data.*

### 4. The incident — the half nobody has asked about

> "On the SNS credential — the part we'd want to rule out for you isn't the SMS. It's whether that same credential could reach S3. Were CloudTrail **data events** enabled on the patient buckets at the time?"

*S3 object-level data events are **off by default**; CloudTrail logs management events only unless you turn data events on explicitly. If they were off, HippoClinic cannot demonstrate that PHI was not read during the compromise window — and under the Breach Notification Rule an incident of unknown scope needs a documented risk assessment, not an assumption. This is the single most valuable observation available on this call. It is also the one that most clearly justifies paying someone to look properly.*

Say the useful part out loud: *"If they weren't on, that's worth fixing this week regardless of what else we do together — it's a checkbox and it changes what you can say the next time."*

### 5. What's actually irreplaceable?

> "Of the 10 to 20 GB per patient — how much is source data you could never recreate, and how much is derived output from the segmentation and localization pipelines that you could regenerate from the raw signal?"

*The biggest cost and durability lever in the deal, and it is a question about their science rather than their infrastructure, so they will enjoy answering it. Derived artifacts do not need the same protection or the same storage class as the golden source. If a meaningful share of that 10–20 GB is regenerable, the redundancy conversation gets much cheaper.*

Pair it with: **"And after you deliver a diagnosis, how often does anyone read that patient's data again?"** That answer sets the tiering policy.

### 6. Calibrate the money

> "What's the current total footprint and the monthly S3 spend? We can do the tiering math properly, but only against real numbers."

*2 TB/month of growth tells us nothing about whether the bill is $3K a year or $300K. Every cost claim we make later depends on this, and asking for it is how we avoid promising a saving we cannot size. If they will not say on the call, ask for a Cost Explorer export by service — that request alone signals we intend to do arithmetic rather than hand-wave.*

---

## If a seventh question fits

> "Do you have a signed BAA with AWS, and have you checked that every service in the pipeline is HIPAA-eligible?"

*Polite, fast, and occasionally it turns up something. Worth asking mainly because a "yes, of course" from a HIPAA-focused company still sometimes turns out to be "somebody signed something in 2023."*

---

## What to ask for at the end

Do not propose scope. Ask for the inputs that let us propose scope:

1. **A read-only account walkthrough or an architecture diagram** — whatever exists, however rough.
2. **A Cost Explorer export**, grouped by service and storage class, for the last six months.
3. **A 60-minute technical follow-up** with the person identified in question 1, without AWS in the room. Different conversation, more candour.

Then name the shape without pricing it: a bounded review covering durability posture and HIPAA security posture, delivered against their actual accounts, ending in a prioritised remediation plan. AWS has already recommended exactly this, so we are agreeing with their account team rather than competing with it.

---

# TIER 2 — For the technical follow-up, not today

**Security**

- Where did the leaked credential live — repository, CI, a client bundle, a notebook? Was it a long-lived IAM access key, and are long-lived keys still in use anywhere?
- How did you find out — your own alerting, or an AWS abuse or billing notice? *(Answers their detection maturity in one sentence.)*
- Is GuardDuty on? Security Hub? Config? Is CloudTrail centralised to a separate account?
- Was the root cause established, or was the fix rotate-and-disable?
- Who has console access, and is it federated through SSO or individual IAM users with keys?
- Is there any egress control on the accounts, and is PHI encrypted with KMS customer-managed keys or SSE-S3?

**Durability and DR**

- Is there a documented RPO and RTO, and has a restore ever actually been tested?
- Is there any copy of anything outside the two accounts — a different region, a different provider, offline?
- What is the retention obligation, and where does it come from? *(HIPAA sets no retention period for the clinical data itself — six years applies to HIPAA documentation such as policies and BAAs. Actual medical-record retention is state law, and for anything touching a device submission or a trial it is a different regime again. In our experience "we keep everything" is usually a decision nobody made.)*
- Are the buckets versioned, and if so, are old versions being lifecycled or silently accumulating and being paid for?

**Cost**

- Split the bill: storage vs compute vs egress. If clinicians stream video and MRI, transfer may be a larger line than storage.
- Where does the segmentation and localization compute run — EC2 on-demand, Batch, SageMaker? Spot or reserved? GPU instance types?
- Are the Nvidia Inception credits actually redeemable against an AWS bill, or only against Nvidia's own programs? *(Assume the latter until shown otherwise.)*
- What is the cost per patient per year, and how does it move against revenue? If they charge per clinician seat and cost scales per patient, the unit economics deteriorate as they grow — which is the real reason storage cost "feels" like it is climbing fast.
- Have they modelled Savings Plans or Reserved Instances on the compute?

**Platform and roadmap**

- What does the ingestion path look like end to end, from acquisition device to S3?
- Is the platform multi-tenant, and how is one clinic's data isolated from another's?
- What is the actual engineering headcount, and what would they want us to own versus advise on?
- Are they pursuing SOC 2 or HITRUST, or is HIPAA the only compliance surface?

---

# The five-minute version

If the call collapses to introductions and you get one exchange:

1. Single copy — S3 Standard with versioning off, or One Zone-IA?
2. Were CloudTrail data events on the patient buckets during the SNS incident?
3. Two accounts — Organization with SCPs, or billing only? Is PHI in the same account as the app?
4. Total footprint and monthly S3 spend?
5. Who owns the AWS estate day to day, and can we get 60 minutes with them?

---

## Two things not to do

**Don't sell an architecture.** We have one Slack message. Any diagram we bring today is a guess, and a wrong guess in front of the AWS account team is expensive.

**Don't lead with the incident.** Open with the durability reframe, get them agreeing that we understand the problem, and let the CloudTrail question land inside that goodwill rather than ahead of it.
