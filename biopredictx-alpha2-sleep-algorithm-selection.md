---
type: Note
related_to:
  - "[[biopredictx-alpha-2-0]]"
  - "[[biopredictx-alpha2-huggingface-model-approach]]"
  - "[[biopredictx-alpha2-risks]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — Sleep-Staging Algorithm Selection (Garmin Vivoactive 5)

Deep-research findings on selecting a **near-real-time (online)** sleep-stage algorithm driven by **Garmin Vivoactive 5** real-time SDK streams, to trigger Restiv therapy **before** deep-sleep onset. Validation reference is a Muse EEG worn in parallel. Related to [[biopredictx-alpha-2-0]].

## Bottom line

Online deep-sleep-onset detection from a Vivoactive 5 is **feasible to attempt but scientifically unproven for this exact use case** — it is the single biggest risk in the program (ties to R1 / OQ-11 in [[biopredictx-alpha2-risks]]). Prototype it first; treat it as a hypothesis to test, not a component to spec.

## What the Vivoactive 5 gives in real time

Via the Garmin Health SDK (Companion/Standard): heart rate, enhanced beat-to-beat intervals (HRV), accelerometer, respiration rate, Pulse Ox, stress. **No real-time sleep stages** (sleep is a post-sync batch metric) and **no EEG**. So the stager must be built on cardiac + actigraphy + respiratory signals.

## Accuracy of non-EEG staging vs EEG/PSG — and the deep-sleep problem

Consistent across independent sources: non-EEG staging reaches ~77% four-class accuracy / Cohen's κ ≈ 0.6, but **deep sleep (N3) is the worst-performing stage and is systematically underestimated**.

- **Sridhar et al., npj Digital Medicine 2020** (heart-rate-only deep learning, >10,000 nights): 77% acc / κ=0.66 held-out; **deep-sleep recall only 0.49** (catches ~half of deep sleep); explicitly states it "cannot be used for real-time sleep stage detection, as the model requires a full night of data."
- **Radha et al., Scientific Reports 2019** (HRV-LSTM, 292 participants): κ=0.61 ± 0.15, 77% acc; declines age 50+. LSTM is causal-capable but was validated whole-night.
- **Schyvens et al., JMIR mHealth 2024** (Garmin-specific systematic review): Garmin Vivosmart 4 only "moderate" stage accuracy; its deep-sleep sensitivity sits below a competing device's 75%.
- **Alhejaili et al., Sleep Medicine 2026**: consumer wearables show "generally weak" PSG agreement for stage durations; suitable for "longitudinal self-monitoring rather than clinical-grade assessment of sleep architecture."

## The decisive issue — online / causal

The strong published accuracies are from **non-causal, whole-night** models. Our trigger must decide live, before deep sleep, with no future data. Expect a causal penalty. **Favorable lever:** the FDC's "early low-confidence trigger beats a late high-confidence one" lets us bias the model toward high deep-sleep sensitivity / lead time at the cost of more false early triggers — turning the weakest metric into a tunable knob (must be validated, not assumed).

## Open-source options

- **Walch sleep_classifiers** (MIT, accel + PPG-HR) — usable pipeline, but only wake/NREM/REM; **no deep-sleep separation**.
- **YASA** — best-known open stager, but **EEG-based** → not applicable (no EEG on Vivoactive).
- **SleepECG** (BSD-3) — ECG/HR baseline.
- Sridhar/Radha architectures — reference designs to reimplement causally.
No drop-in, MIT, online, deep-sleep-capable wrist model exists → reimplement a causal HRV/HR + accelerometer model, pretrain on SHHS/MESA/CinC, fine-tune on Garmin↔Muse data.

## Recommendation for the PoC

Build a **causal, deep-sleep-biased HRV/HR + accelerometer classifier** and validate its **lead time to deep-sleep onset** (not just κ) against Muse EEG. Agree the acceptance bar up front (ties to OQ-06/OQ-08): trigger fires before N3 onset on ≥X% of cycles, median lead ≥Y s, ≤Z false triggers/night. State plainly: the PoC may show reliable pre-deep-sleep triggering from a Vivoactive 5 is **not achievable** — which is why it belongs in a paid, gated PoC before any full build.

## Source of truth

Full cited report in the engagement workspace: `Sol — Sales Engineer/engagements/biopredictx-alpha-2-0/research/sleep-algorithm-selection.md`. Companion analysis: [[biopredictx-alpha2-huggingface-model-approach]].
