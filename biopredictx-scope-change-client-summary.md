---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-scope-requirements-comparison]]"
  - "[[biopredictx-tshirt-sizing]]"
status: Active
---

# BioPredictX — Scope Change Summary (Original → Alpha 2.0)

A plain-language view of how the scope changed between the original production plan and Alpha 2.0, written for a client/stakeholder audience. Feature-level, not requirement-level. Sizes are relative effort, not commitments.

**Size legend:** XS = days · S = ~1–2 weeks · M = ~2–4 weeks · L = ~1–2 months · XL = 2+ months · **?** = depends on a feasibility check not yet done.

## The most significant changes

**From a mass-market product to a controlled lab test.** The original plan was a commercial platform for up to 500,000 devices with a path to FDA clearance. Alpha 2.0 is a small, controlled feasibility study to prove the Restiv prototype actually works. Everything below follows from that shift in purpose.

**From "collect now, analyze later" to "act in the moment."** This is the single biggest change. The original uploaded a whole night of sleep data in bulk and analyzed it afterward to tune future therapy. Alpha 2.0 has to watch sleep *in real time* and switch the therapy on *before* the user enters deep sleep. This is a brand-new capability — and the hardest, riskiest part of the build.

**From a patient app to a staff dashboard.** The original built consumer iOS/Android apps. Alpha 2.0 has no consumer app at all — just an internal web portal your researchers use to run tests, configure the device, and export data.

**From custom AI to off-the-shelf.** The original built proprietary machine-learning models in the cloud. Alpha 2.0 leans on commercial/off-the-shelf sleep tracking (Garmin first) to minimize custom software; the proprietary algorithm is deferred to a later Beta.

**From automated compliance to manual controls.** The original engineered a full HIPAA/GDPR compliance machine (secure identity vaults, audit trails, data-erasure workflows). Alpha 2.0 relies on de-identified IDs and manual administrative process, because it's a small test — far less to build, but not production-grade privacy.

**Concrete local hardware.** The original kept hardware abstract and out of scope. Alpha 2.0 names the exact rig: a Raspberry Pi and Arduino controller driving the Restiv device, a Garmin watch feeding it data, and a Muse EEG headband for validation.

## Features, status, and sizing

"Status" tells you whether each item is **New** (did not exist in the original scope), **Reshaped** (existed but is now much lighter), or **Existing** (already built — reused, not developed in this phase).

| Feature (what it does) | Status | Size | What it means for you |
| :-- | :-- | :-: | :-- |
| Real-time therapy trigger — switch therapy on before deep sleep | New | L ? | The heart of the whole test, and the riskiest piece. Depends on detecting the sleep transition fast enough. |
| Real-time sleep detection / analytics | New | XL ? | Replaces after-the-fact analysis. Hardest technical item; may require the EEG headband (Muse) rather than Garmin to work in real time. |
| Live sensor streaming from the wearable to the local box | New | L ? | Replaces the nightly bulk upload. Needs a proven live link from the watch to the Raspberry Pi. |
| Garmin watch app + watch→gateway link | New | L ? | Not written into the spec, but required to get live data off the watch. We flagged it; feasibility is not yet proven. |
| Local hardware gateway integration (Raspberry Pi + Arduino + device) | New | M–L | Wiring the named hardware together. The device's own firmware already exists, so this is integration, not rebuilding it. |
| Internal Operations Portal (staff web dashboard) | New | M | Where your team registers test users, sets therapy parameters (magnet angle, motor speed, frequency), and exports data. |
| API / services backbone | New | M | The plumbing tying the portal, analytics, storage, and device together. |
| Data storage (raw + processed data, sleep stages, therapy logs) | Reshaped | S | Kept, but simplified — tiered local/cloud storage, without the original's heavy compliance vault. |
| Privacy & access control | Reshaped | S | De-identified user IDs plus manual admin controls, instead of the automated HIPAA/GDPR machine. |
| Restiv wellness device + firmware | Existing | — | Reused as-is. You already have it; we integrate with it rather than rebuild it. |
| Therapy mechanism (magnetic-field sleep therapy) | Existing | — | Unchanged from the original product concept. |

## What the original had that Alpha 2.0 does *not* (deferred or dropped)

Worth being explicit, so there's no surprise about what this phase is *not* delivering:

- **Consumer mobile app (iOS/Android)** — deferred to Beta. The portal is built on a single codebase to make that transition easier later.
- **Proprietary predictive AI** — deferred to Beta. Alpha uses off-the-shelf sleep staging with a clean seam for the future algorithm to plug into.
- **Automated HIPAA/GDPR compliance, data-erasure, audit logging** — dropped for the test; replaced by manual controls.
- **500,000-device scale, disaster recovery, uptime guarantees** — dropped; Alpha is a minimal-use test regime.
- **Over-the-air firmware / fleet management** — out of scope.

## Where the cost and risk concentrate

Most of Alpha 2.0 is small-to-medium, predictable work: the staff portal, storage, access control, and hardware integration. The budget's uncertainty lives almost entirely in one cluster — the **real-time sense-to-therapy loop** (live streaming, real-time sleep detection, the therapy trigger, and the Garmin data path). Those carry a **?** because their size depends on two feasibility checks we recommend running first: whether the therapy can realistically fire *before* deep sleep, and whether a real-time link from the Garmin watch to the Raspberry Pi is achievable. If the answers push you toward the EEG headband, that one decision reshapes the estimate more than anything else on this list.
