# Evening Call Summary — Day 4

**Date:** 2026-05-08
**Participants:** Chalie Lijalem & Yonas Eshete

---

Yonas confirmed the binomial resampling analysis was the missing piece — he could now read p=0.372 as the absence probability of the critical pair rather than a small-n artifact, and the 6.8% upper bound as a resampling artifact rather than a plausible effect size. Chalie confirmed the two-path breakdown landed — she could now identify `scoring_evaluator.py:283` (Path 1) and `generate_preference_pairs.py:12` (Path 2) as the specific places where self-preference survives rotation, and understood that the bias inflates absolute scores but not relative deltas, making Delta A robust while absolute tone claims remain qualified. Chalie added the runnable bootstrap script after Yonas noted the binomial argument was clear but had no output reproducing his exact numbers; Yonas noted the stratification script in the received explainer was already runnable against `ablation_results.json` without additional API calls. Both grounding commits were updated the same evening.
