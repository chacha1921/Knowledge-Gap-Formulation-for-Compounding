# Sign-off — Day 4

**Asker:** Chalie Lijalem
**Explainer written by:** Yonas Eshete
**Gap-closure judgment:** Closed

---

Before this explainer I understood cross-family rotation as an evaluation integrity guard without knowing what layer it actually operates on — I could not have told you that it closes a data contamination path (preference leakage) but leaves an inference-time path (self-preference) fully open. I now understand that these are two distinct mechanisms: leakage is about training-data overlap between judge and generated content, and self-preference is about the judge's embedded aesthetic preferences operating at scoring time regardless of model family. More specifically, I now see that my pipeline has two live self-preference paths: Path 1 is `evaluation/scoring_evaluator.py:283` where Gemini evaluates tone and objection-acknowledgment on tasks it also framed in `multi_llm_synthesis.py:15`; Path 2 is the closed loop where Gemini generates chosen preference pairs in `generate_preference_pairs.py:12` and then judges the same tone dimension at held-out time. The most important thing I understand now that I did not before is that self-preference inflates absolute scores but not relative deltas — Delta A is still robust, but any absolute tone-quality claim made from held-out scores is partially confounded and needs the qualifier, and I now know the exact test to run (stratify held-out scores by `generation_model` against `ablation_results.json`) to measure the inflation before making that claim.
