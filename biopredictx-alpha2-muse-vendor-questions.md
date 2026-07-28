---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-muse-athena-eeg-pivot]]"
  - "[[biopredictx-alpha2-phased-scope-muse-athena]]"
  - "[[biopredictx-alpha2-risks]]"
  - "[[biopredictx-alpha2-yasa-viability]]"
status: Active
---

# BioPredictX Alpha 2.0 — Muse (Interaxon) Vendor Technical Questions

Technical due-diligence questions to put to Interaxon / the Muse team to validate that the **Muse S Athena** is a good fit for the Alpha 2.0 EEG-first closed loop. These map directly to the open risks and the Phase 0 spike goals in [[biopredictx-alpha2-phased-scope-muse-athena]] and [[biopredictx-alpha2-muse-athena-eeg-pivot]]. Order matters: the licensing/data-rights answers (Section 1) are gating — if those come back wrong, the rest is moot.

## Why we're asking
The whole architecture depends on being able to (a) legally access and use Athena's raw EEG and derived data, (b) stream it in real time with low, predictable latency into a local closed loop, and (c) run it reliably for a full unattended night on battery. Public spec sheets are engineering-verification items, not answers — we need these confirmed by the vendor before we lock scope.

## 1. SDK access, licensing & data rights (gating — blocks everything downstream)
- What is the exact path to obtain the Muse SDK for the S Athena, and what is the timeline and cost for developer/partner access?
- Does the license permit a **commercial clinical/research product** built on the SDK (not just consumer/research-hobbyist use)? Any per-device, per-seat, or royalty terms?
- Do we have the right to access, store, and process the **raw EEG stream**, and to derive our own features/algorithms from it? Who owns the **derived data and any models** we train on Athena output?
- Are there restrictions on using Athena data to **drive an external therapy device** (Restiv) in a closed loop?
- Any regulatory posture we should know about — is Athena positioned as a wellness device, and does that constrain how we can describe or use it in a clinical feasibility study?
- Is there an NDA / data-processing agreement required before we can evaluate the SDK?

## 2. Real-time streaming, SDK capabilities & tech stack
- What **languages, bindings, and platforms** does the SDK support (iOS / Android / desktop Windows / macOS / Linux)? This decides our gateway stack — see gateway questions below.
- Can the SDK stream **all channels at native resolution in real time** to a host application, or only deliver batched/processed data or post-session files?
- What **processed values** does the SDK expose out of the box (band powers, stage/sleep estimates, signal-quality/horseshoe fit, PPG/fNIRS-derived metrics), and at what cadence?
- What is the **end-to-end streaming latency** from electrode to host callback, and how stable is it? (Our trigger must fire *before* deep-sleep onset — we need to design against a known latency floor.)
- What **timestamping** does the stream provide — device-side timestamps, sample counters, or host arrival time only? What is expected **clock drift** over a full night? (We need <1 s EEG↔therapy sync.)
- Does the SDK / device **buffer on-board** during a BLE dropout and backfill on reconnect, or is data lost during interruptions?
- What is the **reconnection behavior** after a BLE drop — automatic, and how fast?
- Can `Muse Direct` desktop record and replay raw files we can use during the spike before full SDK access is granted?

## 3. Signal specifications (confirm against real firmware, not spec sheet)
- Confirm the **EEG channel count and montage**: 4 primary + 4 amplified auxiliary? Electrode locations (frontal montage) and reference/ground?
- Confirm **sample rate (256 Hz?), bit depth (14-bit?), and units** delivered to the host — raw µV, or scaled/filtered values? Any on-device filtering/notch we can't disable?
- What **accelerometer / gyro** data is available and at what rate (~52 Hz?) for movement/artifact rejection?
- What **PPG and fNIRS** streams are exposed, at what rates and resolution, and are they usable for the later autonomic track?
- What **signal-quality / electrode-contact metrics** does the device/SDK provide per channel, and how should we interpret them for a quality gate?
- On a **frontal montage**, what is the expected quality for **slow-wave / delta detection** overnight? Any vendor guidance on delta amplitude/thresholds for frontal derivations (library defaults assume central derivations)?

## 4. Battery & overnight operation
- What is the **actual continuous runtime** when streaming all channels at native resolution over BLE (not the advertised ~10 h consumer figure)?
- Does the SDK expose **battery-level telemetry** we can log and surface to operators?
- Can the device **stream while charging** (wired overnight) as a fallback if battery can't cover a full session?
- What is the **charge time** from empty, and any battery-health/longevity considerations for nightly use?
- Expected **overnight electrode-contact stability** — does contact/signal degrade over 8–10 h, and are there recommended materials/prep to sustain it?

## 5. Gateway, BLE & connectivity
- What **BLE version and stack** does Athena require, and which host OS/BLE adapters are validated? (We're deciding Raspberry Pi vs powered mini-PC — need the support matrix.)
- Is there a **wired / USB** data path, or is BLE the only real-time transport?
- What is the **maximum reliable range and interference tolerance** for an overnight bedside setup?
- Can **one host manage device identity/pairing** for multiple units (multi-participant study), and is there a device-provisioning/allowlist mechanism?

## 6. Data volume, formats & tooling
- What is the **raw data rate/volume per night** at full resolution (our working allowance is ~100–250 MB/night — confirm)?
- What **file formats** does Muse Direct / the SDK produce for recording and replay, and are they documented/openly parseable?
- Is there a **data dictionary** (channels, units, scaling, quality flags, event markers) we can obtain, or must we reverse-engineer it from SDK output?

## 7. Support, roadmap & continuity
- What **developer support** is available during integration (docs, sample apps, engineering contact, SLA)?
- Is the **S Athena and its SDK a stable, supported product** — any planned deprecations, firmware changes, or breaking API changes on the roadmap?
- Are there **reference customers / case studies** doing real-time SDK streaming or closed-loop control we could learn from?
- What is the **firmware update mechanism**, and can we pin/control firmware versions for study reproducibility?

## How the answers feed the decision
- **Section 1** answers gate the Phase 0 spike — no acceptable licensing/data-rights answer, no build commitment (risk P0-R1, R2').
- **Sections 2–3** confirm whether the *before-deep-sleep* trigger is even physically achievable on this device (risk P1-R1 / R1').
- **Sections 4–5** decide the gateway platform and overnight-reliability design (risks P2-R1…R4, R3'/R4'/R8').
- **Section 6** produces the sensor/data dictionary that is a named Phase 0 deliverable.

See the full risk mapping in [[biopredictx-alpha2-risks]] and the phased plan in [[biopredictx-alpha2-phased-scope-muse-athena]].
