---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha2-phased-scope-muse-athena]]"
  - "[[biopredictx-alpha2-muse-athena-eeg-pivot]]"
  - "[[biopredictx-alpha2-sleep-algorithm-selection]]"
  - "[[biopredictx-alpha2-huggingface-model-approach]]"
status: Active
_organized: true
---

# BioPredictX Alpha 2.0 — YASA Viability Assessment (EEG Sleep Staging)

Deep-research on **YASA** (yasa-sleep.org; Vallat & Walker, *eLife* 2021) as a sleep-staging tool for the **Muse Athena EEG-first** plan. Resolves the "staging-engine fork" open decision in [[biopredictx-alpha2-phased-scope-muse-athena]]. Note: the earlier [[biopredictx-alpha2-sleep-algorithm-selection]] ruled YASA out **only because Garmin had no EEG** — the pivot to Athena changes that, so YASA is now genuinely in play.

## Bottom line

**YASA is viable — but for two of the three roles, not the one everyone worries about.** Adopt it as the **offline overnight staging + deep-sleep-qualification reporting engine**, and as the **validation/ground-truth reference** for benchmarking a custom trigger. Do **not** use YASA as the live "fire therapy before deep sleep" controller: as shipped it is **non-causal** (needs future data), weakest exactly at stage transitions, and trained on a different electrode montage than the Athena. This makes the phased-scope fork concrete: **YASA-as-validation + reporting = yes; the real-time trigger stays custom** (optionally reusing YASA's feature engineering, retrained causally).

## What YASA is

Open-source Python toolbox (BSD-3, `pip install yasa`) built on MNE + LightGBM. Automatic sleep staging from PSG, plus **slow-wave / spindle / REM detection, artefact rejection, spectral analysis, and hypnogram statistics**. Runs **locally** (no cloud — privacy-friendly, matches the pivot's "no internet dependency"); a full night at 100 Hz processes in **<5 s on a basic laptop**. Feature-based (explainable), not a black box.

## Accuracy (eLife 2021, authoritative)

Trained/validated on **>30,000 h / ~3,000+ full-night PSG** recordings (NSRR), heterogeneous ages/health.

- Overall **median accuracy 87.5%, Cohen's κ 0.819** (585-night NSRR holdout) — matches human inter-scorer agreement (~83%).
- **Deep sleep (N3): sensitivity 83.2%, median F1 0.835.** On the independent DOD sets, N3 F1 = 0.87 (healthy) and 0.78 (OSA) — and YASA **beat** the other algorithms and 4/5 human scorers for N3 in OSA patients. **This is the opposite of the Garmin/wrist story: EEG detects deep sleep well.**
- **Single-channel EEG** still gives median accuracy **85.1% (κ 0.782)**; one EEG + one EOG → 86.9%.
- Per-epoch **confidence scores** available; high-confidence epochs reach ~95% accuracy — usable directly as the pivot's quality/confidence gate.
- N1 is weak (45%) — expected, and not our concern (deep sleep is).

## The three blockers for real-time triggering

1. **Non-causal by design.** Feature extraction applies a **"centered 7.5-min triangular-weighted rolling average"** — it uses ±3.75 min of *future* signal — and the paper states performance degrades on data shorter than 7.5 min. The classifiers were trained "exclusively on full-night recordings." You cannot fire *before* an onset that the model only recognizes with future context.
2. **Weakest exactly at transitions.** Accuracy is ~**69% around stage transitions vs ~94% during stable epochs.** Deep-sleep *onset* is a transition — YASA's worst moment is precisely the one the therapy trigger cares about.
3. **Montage mismatch.** YASA's default classifiers prefer a **central** electrode (C4-M1, C3-M2). The **Muse Athena is frontal (AF7/AF8) + ear (TP9/TP10)** — off-label. Frontal EEG is actually favourable for slow-wave/N3 (slow waves are frontally dominant), so N3 may hold up, but this is **unvalidated on Athena** and must be tested. YASA supports **custom classifiers**, so retraining on an Athena montage is possible.

## Fit by role

| Role in the plan | YASA fit | Notes |
|---|---|---|
| **Offline overnight staging (Phase 1 baseline nights)** | ✅ Strong | 87.5% acc, N3 F1 0.835, local, free. Validate on Athena montage first. |
| **Deep-sleep qualification reporting (Phase 3)** | ✅ Strong | Built-in slow-wave detection → cycles, per-cycle duration, slow-wave amplitude/depth the client asked for. |
| **Validation / ground-truth reference for the custom trigger** | ✅ Strong | Pragmatic offline reference to benchmark the real-time model against (still spot-check with an expert per YASA's own caveat). |
| **Real-time before-deep-sleep controller** | ✗ Not as shipped | Non-causal + transition-weak + off-montage. Would require a causal reconfiguration + retraining = custom work, not "run YASA." |

## Recommendation (resolves the staging-engine fork)

- **Adopt YASA now** for offline staging + deep-sleep reporting + as the validation reference. It is free, local, explainable, EEG-native, and strong at N3 — a clear win for the Athena architecture and for the client's expanded reporting.
- **Keep the live trigger custom and causal.** Optionally reuse YASA's documented feature set (Hjorth, fractal dimension, permutation entropy, band-power ratios) in a **past-only/causal** model retrained on Athena frontal data — faster than from scratch, but it is a build, not a download.
- **Validate YASA on the Athena montage** during Phase 0/1 before relying on its numbers (compare against expert scoring or a synchronized reference). Reconciles Lindsey's "YASA" email with Docs 1 & 2's "custom engine": **YASA for offline/reference/reporting; custom causal model for live control.**

## Sources

1. YASA docs — Installation / overview (features, deps, BSD-3, local): https://yasa-sleep.org/index.html
2. YASA docs — `yasa.SleepStaging` API (central-electrode requirement; centered 7.5-min smoothing = non-causal; 30-s epochs; 100 Hz downsample; confidence via `predict_proba`; trained on ~3,000 NSRR full nights): https://yasa-sleep.org/generated/yasa.SleepStaging.html
3. Vallat R & Walker MP (2021), "An open-source, high-performance tool for automated sleep staging," *eLife* 10:e70092 — 87.5% acc / κ 0.819; N3 sensitivity 83.2% / F1 0.835; single-channel 85.1%; transition vs stable 69% vs 94%; non-causal smoothing; BSD-3. https://elifesciences.org/articles/70092

*Methodology note: automated web-search unavailable this session; YASA site pages and the eLife article were read directly via browser and cross-checked. Full copy in the engagement workspace: `Sol — Sales Engineer/engagements/biopredictx-alpha-2-0/research/yasa-viability.md`.*
