# Portfolio Update — Week 12 Grounding Commits

**Trainee:** Chalie Lijalem
**Program:** TRP1 Forward Deployed Engineer — 10 Academy / Tenacious Intelligence Corp

---

Week 12 produced five concrete edits to systems shipped in Weeks 3, 10, and 11. Each edit is traceable to a specific mechanism understood during the paired knowledge-transfer exercise. Together they demonstrate the ability to ship under pressure, identify what the shipped code assumed but lacked, and commit precise improvements rather than rewrites.

| Day | File Edited | What Changed |
|-----|-------------|--------------|
| 1 | `week10/Technical Challenge/baseline.md` | Added Cost Interpretation section naming O(n²) context growth as the mechanism behind the 12x cost spread — previously reported as unexplained variance |
| 2 | `src/strategies/extractors.py` (Document-Intelligence-Refinery) | Added `smart_bbox_crop()` — 15-line OpenCV crop to content bounding box before VLM tiling; reduces tile count ~40–50%, reduces tile-boundary table failures |
| 3 | `training/train_orpo.py` (Sales-Agent-Evaluation-Bench) | Changed LoRA rank r=32 → r=16 with inline justification citing intrinsic dimensionality argument; code now documents a principled choice, not a tutorial default |
| 4 | `tenacious-bench/methodology.md` + `model_card.md` | Added self-preference bias section naming two live paths (Gemini judges tasks it authored; Gemini generates and judges tone dimension); qualified absolute tone claims while preserving Delta A result as robust |
| 5 | *(Day 5 — to be added)* | |

**Why the edits are targeted, not rewrites.** A comment update, a 15-line helper, a model card qualification — none disturb the surrounding system. The precision is the point. An FDE who can diagnose and patch a specific mechanism is safer to deploy on client engagements than one who rewrites from first principles when confused. The grounding commits make that diagnostic capacity visible and verifiable.
