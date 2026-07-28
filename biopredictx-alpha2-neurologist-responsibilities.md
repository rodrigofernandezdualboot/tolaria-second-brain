---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-phased-scope-muse-athena]]"
  - "[[biopredictx-alpha2-muse-athena-eeg-pivot]]"
  - "[[biopredictx-alpha2-risks]]"
status: Draft
---

# BioPredictX Alpha 2.0 — Neurologist / Sleep-Science Responsibilities by Phase

What the client-side neurologist (or sleep-science clinician) must do in each phase of the [[biopredictx-alpha2-phased-scope-muse-athena|Muse Athena EEG-first phased plan]]. This role is not optional: the plan flags **"no in-house sleep-science expertise"** as a standing risk (P1-R2 / R10'), and a **named clinical reviewer whose sign-off is a phase exit gate** is the mitigation. Their core job is to own the answer to *"what does correct look like?"* for sleep state, timing, and safety — decisions engineering cannot make.

## Cross-cutting responsibility (all phases)

Be the **named, accountable clinical/sleep-science reviewer** for the study. Uphold the two scientific-caution rules from the plan (Open Decision #7):
- Do **not** design the controller to "chase" the EEG slow-wave frequency — Restiv rotation frequency and EEG slow-wave frequency are different physical variables; any relationship is a testable hypothesis, not a design target.
- Do **not** treat all low-frequency (delta) power as restorative deep sleep.

## Phase 0 — Athena technical spike (light clinical input)

Mostly engineering, but the neurologist should:
- **Define the physiological targets** the wave-feature spike must detect — which delta / slow-wave characteristics on a **frontal montage** actually indicate emerging deep sleep.
- **Sanity-check the "act before deep sleep" premise** and give a first opinion on a plausible **lead time**, so the latency/threshold feasibility memo is measured against a clinically meaningful target rather than an arbitrary one.
- **Review the re-tuned frontal-montage thresholds** — library defaults are calibrated for adult central derivations, not the Athena frontal montage.

## Phase 1 — Sleep-state engine & therapy decision, shadow mode (heaviest clinical load)

This is where the neurologist is indispensable; their sign-off **is** the exit gate.
- **Define and sign off the custom sleep-state definition** — what "emerging deep sleep" means in EEG feature/event terms.
- **Define the timing requirement** (Open Decision #2): required **lead time before deep-sleep onset + tolerance**. This is a prediction problem, not classification.
- **Decide the trigger signature and threshold** (Open Decisions #3 / #6): which combination of delta/slow-wave power trend and slow-wave event rate/amplitude fires the decision.
- **Weigh in on the staging-engine approach** (Open Decision #1): open-source EEG library as the engine, a proprietary feature-state engine, or the library as validation reference only.
- **Act as the validation reference** — review shadow-mode recommendations against synchronized EEG+Restiv records; confirm signal quality, latency, and classification are clinically acceptable.
- **Guard the science:** ensure low-frequency artifact / poor overnight contact is not mistaken for delta (P1-R5), and that device-derived stages are treated as reference, not ground truth (P1-R4).
- **Renegotiate the lead-time/tolerance** if "before onset" proves infeasible on the frontal montage (P1-R1 contingency).

## Phase 2 — Gateway robustness & operator view (light clinical input)

Primarily an engineering-hardening phase, but the neurologist should:
- **Set the clinical bar for "usable" overnight data** — what EEG completeness, contact quality, and sync tolerance are clinically acceptable (feeding the ≥95% completeness / <1 s sync / recovery targets).
- **Advise on contact-degradation limits** — how much late-night electrode degradation still yields scientifically valid data before a night should be discarded.
- **Approve any fallback** to a shorter session window or wired/charging setup if battery/contact can't sustain a full night (P2 contingency).

## Phase 3 — Controller, portal & restricted activation (safety sign-off, mandatory)

Live therapy on a sleeping participant — clinical sign-off is required **before** any restricted-active night.
- **Sign off the full control logic** (Open Decision #3, currently "to be defined — clinician input required"): modes, allowed actions (start / maintain / pause / resume / taper / stop), quality gates, lockout/dwell, and **hard exposure limits**.
- **Define the safety envelope** for a sleeping participant — set the hard limits and the fault → safe-state (therapy OFF) behavior.
- **Gate restricted activation:** confirm the Phase 1/2 gates passed and give explicit go/no-go before the device fires live (mitigates P3-R1, the highest-consequence risk).
- **Validate the expanded reports** — confirm time-per-stage and deep-sleep-qualification outputs (cycles, per-cycle duration, slow-wave amplitude/depth) are clinically sound.
- If sign-off isn't ready, the plan keeps the device in **shadow / live-acquisition-only** mode and defers activation to a change request — the neurologist's decision gates this.

## Quick reference

| Phase | Clinical load | Must deliver |
| --- | --- | --- |
| 0 | Light | Physiological targets for detection; first lead-time opinion; review frontal thresholds |
| 1 | **Heaviest** | Signed-off sleep-state definition, lead-time/tolerance, trigger signature; act as validation reference |
| 2 | Light | Clinical "usable data" bar; contact-degradation limits; approve any session fallback |
| 3 | **Mandatory safety** | Sign off control logic + exposure limits; go/no-go for live activation; validate reports |
