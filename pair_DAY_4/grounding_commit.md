# Grounding Commit — Day 4

**Asker:** Chalie Lijalem
**File edited:** `tenacious-bench/model_card.md` and `tenacious-bench/methodology.md`

---

## What Changed

Updated the evaluation integrity section of `methodology.md` to distinguish what cross-family rotation protects against from what it does not, and updated `model_card.md` to qualify any absolute tone-quality claims.

**methodology.md — updated evaluation integrity section:**

```
### Evaluation Integrity

**Preference leakage** is addressed via cross-family JUDGE_ROUTING
(`generation/judge_filter.py:72-81`), covering 94% of tasks. If Gemini
generated a task, a non-Gemini judge scores it, and vice versa.

**Self-preference bias** is NOT closed by cross-family rotation. Two live
paths remain:
- Path 1: Gemini evaluates tone and objection-ack on tasks it also authored
  via multi_llm_synthesis.py (evaluation/scoring_evaluator.py:283).
- Path 2: Gemini generates chosen preference pairs (generate_preference_pairs.py:12)
  and also judges tone on the held-out partition — a closed self-preference loop
  on the tone dimension.

Self-preference inflates absolute scores, not relative deltas. Delta A is
robust. Absolute tone-quality claims require the qualifier below.

Pending audit: run stratification script against held-out generation_model
field and ablation_results.json. If Gemini-authored tasks score materially
higher under Gemini evaluation than DeepSeek-authored tasks, that gap
quantifies the inflation. See preference_leakage_memo.md:41-42.
```

**model_card.md — tone quality qualifier added:**

```
Note: held-out tone scores are partially confounded by self-preference bias
on Gemini-authored tasks (Path 1: evaluation/scoring_evaluator.py:283;
Path 2: generate_preference_pairs.py:12 → tone scoring). Delta A (+2.27pp)
is unaffected — self-preference inflates absolute scores equally across
conditions. Absolute tone-quality claims should not be made from held-out
scores until the generation_model stratification audit runs.
```

---

## Why It Changed

Before this explainer the methodology documented cross-family rotation as the evaluation integrity mechanism without distinguishing what it protects from what it leaves open. Yonas's explainer identified two specific code paths — `scoring_evaluator.py:283` and `generate_preference_pairs.py:12` — where Gemini is active at both the generation and evaluation end of the same dimension, creating a self-preference loop that cross-family rotation was never designed to close. The critical distinction is that this inflation is asymmetric by claim type: Delta A is robust because baseline and adapter are scored by the same judge under the same bias, but any absolute tone-quality assertion uses scores that are confounded. Documenting both the live paths and the scope of their impact — rather than a generic "self-preference not tested" disclaimer — is the precise qualification an external reviewer or client would need to interpret the results correctly.
