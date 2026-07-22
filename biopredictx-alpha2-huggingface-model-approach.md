---
type: Note
related_to:
  - "[[biopredictx-alpha-2-0]]"
  - "[[biopredictx-alpha2-sleep-algorithm-selection]]"
  - "[[biopredictx-alpha2-risks]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — Hugging Face Model Approach for Sleep Staging

Deep-research on whether we can change approach and **run an existing Hugging Face model** (local or cloud) to classify sleep phases, instead of building a stager from scratch. Related to [[biopredictx-alpha-2-0]] and [[biopredictx-alpha2-sleep-algorithm-selection]].

## Bottom line

There is **no drop-in HF model that classifies sleep from a Garmin Vivoactive 5's real-time streams.** Every ready-to-run sleep-staging model on the Hub is **EEG-based** (the Vivoactive has no EEG), and the single **cardio-respiratory** model (`wav2sleep`) is **non-causal (whole-night)** and expects **PSG-grade raw waveforms + respiratory belts**, not Garmin's derived HR/HRV/accelerometer. HF is useful as a source of **pretrained architectures to fine-tune**, not as a plug-and-play classifier. The deep-sleep feasibility risk (R1) is unchanged.

## What's on the Hub (`sleep-staging` tag, full enumeration)

- **braindecode/** (USleep, DeepSleepNet, AttnSleep, ContraWR, BIOT, Chambon2018, Blanco2020): all **EEG**; model cards are architecture definitions with **no packaged weights**.
- **4rooms/physioex** (Apache-2.0): SeqSleepNet/SleepTransformer/TinySleepNet/Chambon checkpoints; **EEG / EEG+EOG+EMG**; test acc 0.84–0.885.
- **max-miller/circadia-sleep-student** (Apache-2.0): **EEG** (Sleep-EDF), ONNX, edge-deployable.
- **Feng613/SleepVLM-3B** (Apache-2.0): vision-language model over **EEG/EOG/EMG epoch images**.
- **joncarter/wav2sleep** (MIT): **cardio-respiratory** (ECG, PPG, ABD, THX), 4-class incl. Deep — the one non-EEG model.

## Why wav2sleep still doesn't drop in

1. **Non-causal (bidirectional)** per its own model card — needs the whole night; can't make a live "deep sleep imminent" call.
2. **Wrong input granularity** — expects raw ECG/PPG waveforms (~34 Hz) + respiratory effort belts (ABD/THX); Garmin gives derived beat-to-beat intervals + a respiration rate, and wav2sleep doesn't use the accelerometer.
3. **PSG-trained** on clinic-grade inputs; unvalidated (and expected to degrade) on watch-grade wrist signals.

## Does the HF approach change build vs buy?

Partially and usefully: it moves the PoC from **train-from-scratch** to **fine-tune-from-pretrained** (seed from `wav2sleep` or `physioex`, adapt the input head to Garmin signals, fine-tune on Garmin↔Muse-EEG data). Still requires a **causal retrofit** (the key, accuracy-costing change) and adding the accelerometer. Deployment (ONNX locally, or HF Inference Endpoints in a VPC — subject to the OQ-15 data-posture decision) is a late, low-risk step.

## Recommendation

Keep the plan to build a **causal cardio-respiratory classifier**, but **seed it from Hugging Face** rather than from zero. Benchmark lead-time-to-deep-sleep-onset against Muse EEG. Tell the client plainly: "just run a Hugging Face model" does **not** shortcut the hard part — it modestly de-risks/speeds the PoC, but the deep-sleep detection feasibility risk still must be proven on real Garmin data before any full-build commitment.

## Source of truth

Full cited report: `Sol — Sales Engineer/engagements/biopredictx-alpha-2-0/research/huggingface-model-approach.md`. Companion analysis: [[biopredictx-alpha2-sleep-algorithm-selection]].
