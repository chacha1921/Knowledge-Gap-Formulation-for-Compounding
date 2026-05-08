# Portfolio Update — Week 12 Grounding Commits

**Trainee:** Chalie Lijalem
**Audience:** FDE hiring manager
**Program:** TRP1 Forward Deployed Engineer — 10 Academy / Tenacious Intelligence Corp

---

## Summary

Week 12 produced four concrete edits to systems shipped in Weeks 10 and 11. Each edit is traceable to a specific gap closed during the paired knowledge-transfer exercise: a partner explained a mechanism, I understood it, and I committed an improvement to the artifact that had assumed that understanding without having it. The table below shows what changed and why it matters for FDE work.

---

## The Four Commits

**Day 1 — `week10/Technical Challenge/baseline.md`**
*Gap closed: KV cache mechanics and O(n²) cost growth in multi-turn agents*

The original baseline.md reported `avg_agent_cost: $0.0199` with no explanation of the 12x cost spread across tasks ($0.0077 to $0.0963). The edit added a Cost Interpretation section that names the mechanism: KV cache does not persist across API call boundaries; each turn re-pays prefill cost on the full accumulated context; cost grows O(n²) in turn count, not with task difficulty. The highest-cost tasks (task_4, task_105) ran more tool-call turns — this is now documented and defensible to a technical reviewer rather than appearing as unexplained variance.

**Day 2 — `src/strategies/extractors.py` (Document-Intelligence-Refinery, Week 3)**
*Gap closed: VLM tiling mechanics and tile-boundary table failure*

Strategy C in the Week 3 Document Intelligence Refinery escalated failed document pages to a VLM with `detail: high`. The original code had no preprocessing before the API call and no prediction of when the VLM would fail on tables. The edit added `smart_bbox_crop()` — 15 lines of OpenCV that crop each page to its content bounding box before tiling. This removes whitespace margins, reducing tile count by ~40–50% on standard scanned documents and centering tables relative to tile boundaries. The mechanism: VLM tiles are 512×512 independent crops with no shared positional encoding across boundaries; a table row straddling a boundary loses row continuity. Cropping reduces that failure mode. Token cost per escalated page drops proportionally.

**Day 3 — `training/train_orpo.py` (Sales-Agent-Evaluation-Bench, Week 11)**
*Gap closed: LoRA rank mechanics and intrinsic dimensionality of behavioral alignment*

The Week 11 ORPO fine-tuning used r=32 and α=32 because the Unsloth tutorial used those values. The edit changed r to 16 and updated the inline comment to document the justification: behavioral alignment tasks have low intrinsic dimensionality (Aghajanyan et al., 2021); suppressing banned phrases is a directional shift in activation space, not a factual rewrite; 381 pairs across 4 alignment dimensions sits at the r=32 ceiling, making r=16 the correct test for whether simpler single-dimension cases converge equally well at lower parameter cost. The code now reads as a principled choice, not a tutorial default.

**Day 4 — `tenacious-bench/methodology.md` and `model_card.md` (Week 11)**
*Gap closed: self-preference bias as a distinct evaluation integrity risk from preference leakage*

The Week 11 tenacious-bench documented cross-family rotation as its evaluation integrity safeguard, with no distinction between preference leakage (data contamination) and self-preference bias (inference-time stylistic preference). The edits added a Self-Preference Bias section to methodology.md naming the two live paths in the pipeline — Gemini evaluating tone on tasks it also authored; Gemini generating chosen preference pairs and judging the same tone dimension at held-out time — and updated model_card.md to qualify that self-preference inflates absolute tone scores but not relative deltas. Delta A (+2.27pp) is documented as robust. Absolute tone-quality claims are qualified pending the cross-judge agreement audit.

---

## What This Shows Cumulatively

Each commit closes a gap between what the code said and what I actually understood. Together they demonstrate: the ability to ship a system under time pressure (Weeks 10–11), identify the understanding the system assumed but lacked (Week 12 questioning), and commit precise improvements rather than rewrites. The edits are targeted — a comment update, a 15-line helper, a model card qualification — not architectural changes. The precision is the point: an FDE who can diagnose and patch a specific mechanism without disturbing the surrounding system is safer to deploy on client engagements than one who rewrites from first principles when confused.
