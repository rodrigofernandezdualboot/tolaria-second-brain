---
type: Note
belongs_to: "[[hippoclinic]]"
related_to: "[[hippoclinic]]"
_width: wide
---

# HippoClinic — HIPAA Self-Assessment Checks

**Purpose:** the questions HippoClinic should put to themselves, mapped onto the estate we actually know about — S3, accounts 6043 and 6269, one copy of everything, and a credential compromise already on the record.

**Not legal advice.** Every regulatory citation below should be confirmed with healthcare counsel or a qualified HIPAA assessor before anyone relies on it. Citations were written from reference knowledge without live verification in this session; treat the specific paragraph numbers as pointers to look up, not as quotations.

---

## Why this document is worth their time

Three of the Security Rule's **Required** implementation specifications land squarely on what HippoClinic has told AWS it is not doing. Required means there is no risk-based opt-out — unlike Addressable specs, where a documented alternative or a documented reason not to implement is acceptable.

| Required spec | Rough citation | What HippoClinic has said |
|---|---|---|
| **Data backup plan** — maintain retrievable exact copies of ePHI | §164.308(a)(7)(ii)(A) | One copy only, redundancy declined on cost grounds |
| **Disaster recovery plan** — procedures to restore any loss of data | §164.308(a)(7)(ii)(B) | Asking AWS for backup/DR *guidance*, so presumably none documented |
| **Audit controls** — mechanisms that record and examine activity in systems holding ePHI | §164.312(b) | Unknown whether CloudTrail data events are on the patient buckets |

Add **information system activity review** (§164.308(a)(1)(ii)(D), also Required) — you must regularly *review* the logs, not merely keep them.

This reframes the cost conversation completely. HippoClinic has been treating redundancy as an expense to defer. If the data is ePHI and they hold a single copy with no documented backup procedure, it is closer to an unmet obligation than a deferred nice-to-have. The cheapest defensible answer is not a second hot copy — it is a versioned, Object-Locked archive-tier copy in a separate account, which costs a fraction of what they fear and satisfies the spec.

**That is the sentence to land: the thing you thought you couldn't afford is the thing you're already required to have, and it costs less than you think.**

---

## Section 0 — The two questions that reframe everything else

### 0.1 Are you a Covered Entity or a Business Associate?

This determines which rules bind you directly, and a surprising number of health-tech SaaS companies have never settled it.

If the ~100 clinicians and their clinics are the Covered Entities, and HippoClinic processes patient data on their behalf, then **HippoClinic is a Business Associate**. Since the HITECH Act and the 2013 Omnibus Rule, Business Associates are **directly liable** under the Security Rule and the Breach Notification Rule, and OCR can enforce against them without going through the clinic.

Ask yourselves:

- [ ] Which are we, and is it written down anywhere?
- [ ] Do we have a signed BAA with **every** clinic or health system sending us patient data? Not most. Every.
- [ ] Do our BAAs commit us to breach notification timelines shorter than the regulatory floor? Many customer-drafted BAAs demand notice in 24 or 48 hours. Do we know what we've promised?
- [ ] Have we signed **downstream** BAAs with every subcontractor that touches PHI — AWS, and anything else in the pipeline: monitoring, error tracking, support tooling, analytics, transcription, a contracted radiologist?
- [ ] If a clinician uploads data for a patient who is not that clinic's patient, whose obligation is it?

**Why this is first:** if HippoClinic is a Business Associate, their customers' auditors will eventually ask these questions on their behalf, and "we follow internal procedures" will not survive that. Getting ahead of it is a sales asset, not just a compliance chore.

### 0.2 Is all of it PHI, and could some of it stop being PHI?

De-identified data falls outside HIPAA. Two routes exist: Safe Harbor removal of the 18 identifiers, or Expert Determination (§164.514). This is the single largest scope-reduction lever available.

- [ ] Which of the three data classes — raw brain signal, video, MRI — is identifiable, and by what argument?
- [ ] **Full-face photographic images and comparable images are one of the 18 Safe Harbor identifiers.** Patient video is plainly in scope. Are we storing video that we no longer clinically need?
- [ ] **Are the MRI volumes defaced or skull-stripped?** A raw structural MRI permits facial reconstruction, which is why defacing is standard practice in neuroimaging data sharing. If we are storing un-defaced volumes for research, secondary analysis or model training, that is identifiable data sitting in a lower-scrutiny part of the pipeline.
- [ ] Does the **DICOM header metadata** still carry patient name, MRN, date of birth, accession number, institution and device serial? Header scrubbing is routinely forgotten because the pixels get all the attention.
- [ ] Are filenames, S3 object keys, or bucket prefixes carrying identifiers? An object key of `patients/mrn-482913/eeg-2026-03-11.edf` leaks PHI into CloudTrail logs, access logs, error messages, support tickets and anywhere a URL gets pasted.
- [ ] For any research or algorithm-development use of this data: is it de-identified, covered by an IRB waiver, or covered by patient authorization? Which one, per dataset?

**Why this matters commercially:** if the archive tier can hold de-identified or defaced data, the compliance surface shrinks along with the bill.

---

## Section 1 — Paper and process (Administrative Safeguards, §164.308)

### Risk analysis and management

- [ ] Have we ever completed a **written risk analysis** of where ePHI lives, how it moves, and what threatens it? (§164.308(a)(1)(ii)(A), Required.) This is the most-cited failure in OCR enforcement actions, and "we follow internal quality and security procedures" is not a risk analysis.
- [ ] Is there a **risk management plan** — documented decisions, owners and dates for the risks the analysis found? (Required.)
- [ ] When was either last updated? A risk analysis that predates the SNS incident is out of date by definition.
- [ ] Is there a named, documented **Security Official**? (§164.308(a)(2), Required.) One person, in writing.
- [ ] Is there a documented **sanction policy** for workforce members who violate the policies? (Required.)

### Access management and workforce

- [ ] Do we know, right now, the full list of humans who can reach patient data, and through what path — console, API keys, database, application admin, support tooling, a laptop with a synced folder?
- [ ] Is access granted on **minimum necessary**, or does everyone in engineering have production read?
- [ ] What happens within 24 hours of someone leaving? Console access, SSO, IAM keys, SSH keys, third-party tools, VPN. Have we ever tested it?
- [ ] Is there **security awareness training** on record, with completion dates, for everyone including contractors and founders?
- [ ] Can a clinician see only their own clinic's patients, and how is that enforced — by application logic alone, or with a data-layer boundary behind it?

### Documentation

- [ ] Do written policies and procedures exist for each safeguard we claim to meet? (§164.316.) Under HIPAA, **the documentation itself must be retained six years** from creation or last effective date. Note that this six-year rule applies to the HIPAA paperwork, not to clinical records — record retention is state law.
- [ ] Where is that documentation, and could we hand a customer's auditor a current copy this week?

---

## Section 2 — The estate (Technical Safeguards, §164.312)

### Audit controls — the gap that matters most today

- [ ] Are **CloudTrail S3 data events** enabled on every bucket holding patient data? They are off by default; CloudTrail records management events only unless data events are switched on explicitly. Without them, there is no record of who read which object.
- [ ] Are **S3 server access logs** on, and are they going somewhere the workload account cannot edit?
- [ ] Is CloudTrail delivered to a bucket in a **different account** with Object Lock, so a compromised principal in 6269 cannot rewrite its own tracks?
- [ ] Does anyone **read** the logs? Required spec §164.308(a)(1)(ii)(D) is about review, not retention. Is there an alert that fires on anomalous bulk read of patient objects, or would a slow exfiltration look like normal Tuesday traffic?
- [ ] Does the application keep its own PHI access log — which clinician opened which patient record, when — separate from infrastructure logs? Infrastructure logs show the service reading S3; they do not show which human asked it to.
- [ ] How long are logs retained, and does that satisfy what our BAAs promise customers?

### Access control and authentication

- [ ] Is **MFA** enforced on every human principal, including the root users of 6043 and 6269? Root MFA on the payer account is the one nobody remembers.
- [ ] Are there any **long-lived IAM access keys** still in existence? The SNS incident was a leaked credential. Where else does that pattern still exist — CI, cron jobs, notebooks, a mobile or web client bundle, a `.env` on someone's laptop?
- [ ] Is there federated SSO with a real identity provider, or individual IAM users?
- [ ] Is there **secret scanning** on the repositories, and did it exist before the incident or because of it?
- [ ] Are there IAM roles with `s3:*` or `s3:DeleteObject` on patient buckets that no longer need them?
- [ ] Is there an **emergency access procedure** — a documented break-glass path for reaching ePHI when normal systems are down? (§164.312(a)(2)(ii), Required.) Break-glass access that exists informally but is not documented fails this twice: it is undocumented, and it is unmonitored.
- [ ] Is there automatic logoff on the clinician-facing application? (Addressable, but clinical workstations are shared.)

### Encryption and integrity

- [ ] Is patient data encrypted at rest with **KMS customer-managed keys**, or with SSE-S3? CMKs give a second, independent policy boundary — a principal needs both bucket permission and key permission. With SSE-S3 there is only one boundary to breach.
- [ ] If CMKs: is the key policy separately administered, or would one compromised principal hold both?
- [ ] Is TLS enforced in transit, including on any internal service-to-service hop and any acquisition-device upload path? Is there a bucket policy denying non-TLS requests?
- [ ] Is there an integrity mechanism — checksums or object-level validation — proving a stored signal file has not been altered? (§164.312(c), Addressable, but for diagnostic source data it is worth doing on clinical grounds regardless.)

**Note on encryption and the breach safe harbour:** encryption is an *Addressable* specification, which sometimes gets read as optional. The practical reason to do it properly is that PHI encrypted to HHS-recognised standards is not "unsecured PHI," and a breach of it generally does not trigger the notification duty. Encryption is the difference between an incident and a reportable breach.

### Account architecture

- [ ] Do the application runtime and the patient data share account 6269? If so, the SNS compromise had reach into the data whether or not it was used that way.
- [ ] Is the payer account 6043 doing governance work — SCPs, centralised logging, an isolated backup destination — or only aggregating invoices?
- [ ] Is there **any** copy of patient data that a compromised principal in 6269 cannot delete? This is the single question that determines whether a bad afternoon is recoverable.
- [ ] Is GuardDuty on? Config? Security Hub? Are the findings routed to a human?
- [ ] Is there egress control, or could a compromised workload stream 2 TB out without anything objecting?

---

## Section 3 — Contingency plan (§164.308(a)(7)) — where they are most exposed

- [ ] Is there a **written data backup plan** producing retrievable exact copies of ePHI? (Required.) One copy in S3 is not a backup; it is the primary.
- [ ] Is there a **written disaster recovery plan** with procedures to restore lost data? (Required.)
- [ ] Is there an **emergency mode operation plan** — how clinical service continues while systems are degraded? (Required.) If a clinician cannot reach a patient's signal data, what happens to that patient's appointment?
- [ ] Has a **restore ever actually been performed**? Not planned. Performed, timed, and written down. (Testing and revision is Addressable, but an untested restore is an assumption, not a control.)
- [ ] Do we have a stated **RPO and RTO**, and do our customer BAAs commit us to numbers we have never measured?
- [ ] Has anyone done the **applications and data criticality analysis** — which systems and datasets must come back first? (Addressable.) For HippoClinic the answer probably distinguishes irreplaceable raw signal from regenerable derived output, which is the same distinction that drives the storage bill.
- [ ] Is **versioning** enabled on patient buckets, so an overwrite or delete is recoverable? Is **MFA delete** or **Object Lock** protecting anything?
- [ ] If versioning is on: are non-current versions being lifecycled, or silently accumulating and being paid for?

**The recommendation this section points to:** a third account whose only job is holding backups, with Object Lock in compliance mode, archive-tier storage, and a role that 6269 cannot assume. It satisfies two Required specs, contains the blast radius of the incident they already had, and costs a fraction of a second hot copy.

---

## Section 4 — Incident and breach readiness (§164.400–414)

The SNS incident is the most useful thing in this whole assessment, because it already happened and can be walked through concretely.

- [ ] Is there a **written incident response procedure**, and was it followed? (§164.308(a)(6), Required.)
- [ ] Was the incident **documented** — timeline, scope, root cause, remediation — or is it institutional memory?
- [ ] Was a **breach risk assessment** performed? The rule sets out four factors: the nature and extent of the PHI involved, who the unauthorized person was, whether PHI was actually acquired or viewed, and the extent to which the risk has been mitigated. A conclusion of "low probability of compromise" has to be reasoned and written, not assumed.
- [ ] **Could we have proven PHI was not accessed?** If S3 data events were off, the third factor is unanswerable. That is the crux: the absence of logs does not mean nothing happened, and OCR will not read it that way.
- [ ] Was the root cause established, or was the fix rotate-and-disable? Where did that credential live, and does that class of exposure still exist?
- [ ] How did we find out — our own alerting, or an AWS abuse or billing notice? If AWS told us, our detection did not work.
- [ ] Do we know our notification obligations and clocks: individuals within 60 days of discovery; HHS within 60 days for a breach affecting 500 or more, or annually for smaller ones; media notice at 500 or more in a state or jurisdiction. And separately, whatever shorter clock our customer BAAs impose.
- [ ] Would we have to notify our **clinic customers** even for an incident with no confirmed PHI access? Read the BAAs — many define "security incident" broadly enough to require it.
- [ ] Has the risk analysis been updated since the incident? (It should have been.)

---

## Section 5 — Beyond HIPAA, and specific to brain data

HIPAA is a floor, not the whole obligation. For a company whose product is neural data, this section may matter more than people expect.

- [ ] **Neural data is now separately regulated in some US states.** Colorado amended its privacy act in 2024 to bring biological and neural data into sensitive-data protections, and California added neural data to the CCPA's sensitive personal information category the same year. Others have followed. These apply on a residency basis and are not displaced by HIPAA for data outside HIPAA's scope — notably any de-identified or research use. Worth asking counsel about explicitly, because it is new and it is aimed squarely at what HippoClinic does.
- [ ] Which **states** are our patients in, and what does each require? State medical-record retention periods, state breach-notification clocks (several are shorter than 60 days), and state-specific rules for sensitive categories.
- [ ] Any patients in the **EU or UK**? Then GDPR Article 9 special-category data, a lawful basis, transfer mechanism, and data subject rights on top of everything above.
- [ ] Are we in **California**? CMIA sits alongside HIPAA with its own penalties.
- [ ] Does anything we do touch **FDA regulation** — is any part of the platform a medical device or clinical decision support tool? If so, design controls, records and cybersecurity expectations arrive from a different direction entirely.
- [ ] Are we doing anything with this data that patients did not consent to — model training, secondary research, benchmarking, a published dataset? Under what authorisation?
- [ ] Are we pursuing **SOC 2** or **HITRUST**? Neither is HIPAA, but enterprise health systems will ask, and the control work overlaps heavily with everything above. Doing it once for three purposes is cheaper than three times.

---

## Section 6 — The regulatory horizon

HHS issued a proposed rule in early 2025 that would substantially tighten the Security Rule — reportedly moving many currently Addressable specifications to Required, and adding explicit requirements around multi-factor authentication, asset inventory, network segmentation, vulnerability scanning and penetration testing, and a defined restoration window for critical systems.

**Verify the current status before planning around it** — proposed rules change materially between NPRM and final, and this session had no live web access to check where it stands. But the direction is clear enough to inform sequencing: controls that are optional today are the likely mandates of the next cycle, and building them now is cheaper than retrofitting them under a compliance deadline.

---

## How to use the answers

Sort every answer into one of four buckets. The distribution is the finding.

1. **Yes, and documented** — the only answer that survives an audit.
2. **Yes, but not written down** — a control that exists as habit. Fails on documentation grounds and evaporates when the person who holds it leaves.
3. **No, and we decided that** — legitimate for Addressable specs, provided the decision and its rationale are written down. Not available for Required ones.
4. **We don't know** — the ones to work on first. Not because they are the worst, but because you cannot manage a risk you cannot see, and every one of these is also an answer you may need to give a customer's auditor under time pressure.

**If only five get answered:**

1. Are we a Covered Entity or a Business Associate, and do we have BAAs in both directions?
2. Is there a written risk analysis, and does it postdate the SNS incident?
3. Is there any copy of patient data that a compromised principal in 6269 cannot delete?
4. Were CloudTrail S3 data events enabled on the patient buckets during the incident?
5. Has a restore ever been performed, timed and written down?

---

## Related

- [[hippoclinic]] — deal context, estate, and our read on the three stated asks.
- [[hippoclinic-call-questions-2026-08-19]] — the 30-minute call question set. Questions 2, 3 and 4 there are the live versions of Sections 2 and 3 here.
