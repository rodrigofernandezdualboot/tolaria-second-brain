---
type: Note
belongs_to: "[[hippoclinic]]"
related_to: "[[hippoclinic]]"
_width: wide
---

# HippoClinic — Question Set for the 30-Minute AWS Sync (19 Aug 2026)

**Call:** Wed 19 Aug 2026, 17:30–18:00 UTC-3. Zoom, hosted by Richard King (AWS).
**In the room:** Richard King (AWS), engineering@hippoclinic.com, Blaine Wagner, Matias Curbelo, Rodrigo Fernandez.
**Prepared from:** Blaine's 12 Aug brief in `#hippoclinic-dealteam-sales` and the AWS meeting recap Blaine forwarded on 19 Aug. No architecture doc, no transcript, no prior technical contact.

---

## How to use this

Thirty minutes with AWS in the room is not a discovery call. Realistically you get **six or seven real questions** and one idea worth remembering. Everything else is introductions and scheduling.

So the goal is narrow:

1. Land **one reframe** that shows we understand their problem better than they stated it.
2. Name **one concrete fix** — because the recap says outright that they are "unclear on what specific steps AWS can take to further secure their platform." They do not know what to ask for. Give them one specific thing and we become the people who answer that.
3. Get the **four facts** that decide whether this is a small review or a real engagement, then leave with a named human and a next meeting.

Do not run the security incident as an interrogation. They already shut a service down over it; they know it went badly. Frame every security question as "here's what we'd want to rule out for you."

**Zua** is the only named person on their side — they raised the durability question with AWS. If Zua is on the call, the reframe below is aimed at them.

---

## The one thing to say before you ask anything

> "Before we get into it — the way the durability question is framed may be doing you a disservice. S3 doesn't lose objects; it stores every one across at least three Availability Zones. What it does do is delete an object the moment something with valid credentials asks it to. Given you've already had a credential get out, we'd look at this as a blast-radius problem rather than a hardware problem — and the fixes are different."

Then the concrete fix, because that is what they said they never get:

> "Specifically: you have two accounts, a payer and a workload account. That means anything that can reach your data can also reach any copy of it, because there's nowhere else for a copy to live. The thing we'd want to build first is a third account whose only job is holding backups — Object Lock on the bucket, and a role your workload account can't assume. That's cheap, and it's the control that would have contained the incident you already had."

And the commercial half:

> "The reason that matters for cost is that you told AWS you're keeping one copy because redundancy is expensive. Our expectation is that most of two terabytes a month is write-once brain signal nobody reads after the diagnosis goes out — and archive tiering on that runs roughly an order of magnitude below what you're paying now. There's a good chance the saving pays for the protection outright. We'd want your actual numbers before promising that, but we don't think you have the trade-off you think you have."

That is the whole pitch. Ninety seconds, technically correct, and nobody has said it to them.

---

# TIER 1 — Ask these today

Six questions. In this order.

### 1. Who's on the call, really?

> "Before we dive in — who owns the AWS estate day to day on your side? Zua, is that you, or is there a platform person we should have in the room next time?"

*`engineering@` is a shared alias and Zua is a first name in a meeting recap. Everything about how to run the next 25 minutes depends on who is actually accountable for the accounts, and the technical champion has to be a person, not a mailbox.*

### 2. When you say a single copy — what exactly is that?

> "Is that S3 Standard with versioning switched off, or is it One Zone-IA?"

*Highest information per second in the set. **S3 Standard with versioning off** means the data is genuinely durable against hardware failure and the exposure is deletion — a governance fix, cheap. **One Zone-IA** means they have taken a real durability reduction: a single Availability Zone, no resilience to losing it. Different conversation, more urgent. If the answer is "I'd have to check," that itself tells you the storage class was never a deliberate decision.*

Follow-on if there is room: is versioning on anywhere, and is there Object Lock or MFA delete on the patient buckets?

### 3. Which account holds what?

> "You've got 6043 as the payer and 6269 as the workload account. Does 6269 hold both the application runtime and the patient data, or is there separation inside it? And is there any SCP boundary between them, or is the link billing-only?"

*We already know the shape — asking "is it an Organization?" would read as not having done the reading. What we do not know is whether the app and the PHI share a blast radius, and whether the payer is doing any governance work or just aggregating invoices. If it is the former in both cases, the SNS compromise was potentially a data incident, and that sets up question 4.*

### 4. The incident — the half nobody has asked about

> "On the SNS credential — the part we'd want to rule out for you isn't the SMS. It's whether that same credential could reach S3. Were CloudTrail **data events** enabled on the patient buckets at the time?"

*S3 object-level data events are **off by default**; CloudTrail logs management events only unless you turn data events on explicitly. If they were off, HippoClinic cannot demonstrate that PHI was not read during the compromise window — and under the Breach Notification Rule an incident of unknown scope needs a documented risk assessment, not an assumption. The single most valuable observation available on this call, and the one that most clearly justifies paying someone to look properly.*

Say the useful part out loud: *"If they weren't on, that's worth fixing this week regardless of what else we do together — it's a checkbox, and it changes what you can say the next time."*

### 5. What's actually irreplaceable?

> "Of the 10 to 20 GB per patient — how much is source data you could never recreate, and how much is derived output from the segmentation and localization pipelines that you could regenerate from the raw signal?"

*The biggest cost and durability lever in the deal, and it is a question about their science rather than their infrastructure, so they will enjoy answering it. Derived artifacts do not need the same protection or the same storage class as the golden source. If a meaningful share of that 10–20 GB is regenerable, the redundancy conversation gets much cheaper.*

Pair it with: **"And after you deliver a diagnosis, how often does anyone read that patient's data again?"** That answer sets the tiering policy.

### 6. Calibrate the money

> "What's the current total footprint and the monthly S3 spend? We can do the tiering math properly, but only against real numbers."

*2 TB/month of growth tells us nothing about whether the bill is $3K a year or $300K. Every cost claim we make later depends on this, and asking for it is how we avoid promising a saving we cannot size. If they will not say on the call, ask for a Cost Explorer export by service and storage class — that request alone signals we intend to do arithmetic rather than hand-wave.*

---

## If a seventh question fits

> "Do you have a signed BAA with AWS, and have you checked that every service in the pipeline is HIPAA-eligible?"

*Polite, fast, and occasionally it turns something up. A "yes, of course" from a HIPAA-focused company still sometimes means "somebody signed something in 2023."*

---

## On credits — say this, don't ask it

They are counting on ~$100K of Nvidia Inception credits and have applied to AWS for more. Worth one sentence:

> "One flag on the Inception credits — those usually apply to Nvidia's own programs rather than an AWS bill. Worth confirming before you plan around them. And either way, a credit offsets a cost that grows per patient; tiering changes the slope. You want both, but only one of them compounds."

---

## What to ask for at the end

Do not propose scope. Ask for the inputs that let us propose scope:

1. **A read-only account walkthrough or an architecture diagram** — whatever exists, however rough.
2. **A Cost Explorer export**, grouped by service and storage class, for the last six months.
3. **A 60-minute technical follow-up** with the person identified in question 1, without AWS in the room. Different conversation, more candour.

Then name the shape without pricing it: a bounded review covering durability posture and HIPAA security posture, delivered against 6043 and 6269, ending in a prioritised remediation plan. AWS has already recommended exactly this, so we are agreeing with their account team rather than competing with it.

---

# TIER 2 — For the technical follow-up, not today

**Security**

- Where did the leaked credential live — repository, CI, a client bundle, a notebook? Was it a long-lived IAM access key, and are long-lived keys still in use anywhere?
- How did you find out — your own alerting, or an AWS abuse or billing notice? *(Answers their detection maturity in one sentence.)*
- Is GuardDuty on? Security Hub? Config? Is CloudTrail centralised anywhere outside 6269?
- Was the root cause established, or was the fix rotate-and-disable?
- Who has console access, and is it federated through SSO or individual IAM users with keys?
- Is there egress control on the accounts, and is PHI encrypted with KMS customer-managed keys or SSE-S3? If CMKs — is the key policy separate from the bucket policy, or would one compromised principal get both?

**Durability and DR**

- Is there a documented RPO and RTO, and has a restore ever actually been tested?
- Is there any copy of anything outside the two accounts — a different region, a different provider, offline?
- What is the retention obligation, and where does it come from? *(HIPAA sets no retention period for the clinical data itself — six years applies to HIPAA documentation such as policies and BAAs. Actual medical-record retention is state law, and anything touching a device submission or a trial is a different regime again. In our experience "we keep everything" is usually a decision nobody made.)*
- Are the buckets versioned, and if so, are old versions being lifecycled or silently accumulating and being paid for?

**Cost**

- Split the bill: storage vs compute vs egress. If clinicians stream video and MRI, transfer may be a larger line than storage.
- Where does the segmentation and localization compute run — EC2 on-demand, Batch, SageMaker? Spot or reserved? Which GPU instance types?
- What is the cost per patient per year, and how does it move against revenue? If they charge per clinician seat while cost scales per patient, unit economics deteriorate as they grow — which is the real reason storage cost "feels" like it is climbing fast.
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
3. Does 6269 hold the app runtime and the patient data together?
4. Total footprint and monthly S3 spend?
5. Who owns the AWS estate day to day, and can we get 60 minutes with them?

---

## Two things not to do

**Don't sell an architecture.** We have two secondhand summaries. Any diagram we bring today is a guess, and a wrong guess in front of the AWS account team is expensive. The backup-account recommendation is the exception — it is one control, not a design.

**Don't lead with the incident.** Open with the durability reframe, get them agreeing that we understand the problem, and let the CloudTrail question land inside that goodwill rather than ahead of it.
