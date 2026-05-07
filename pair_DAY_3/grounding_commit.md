# Grounding Commit — Day 3

**Asker:** Chalie Lijalem
**File edited:** `training/train_orpo.py` in [Sales-Agent-Evaluation-Bench](https://github.com/chacha1921/Sales-Agent-Evaluation-Bench)

---

## What Changed

Changed LoRA rank from r=32 to r=16 in `LORA_CONFIG` and re-ran training to compare held-out performance against the r=32 baseline (4.462/5.0).

**Before:**
```python
LORA_CONFIG = dict(
    r=32,
    lora_alpha=32,  # unsloth: alpha == r
    ...
)
```

**After:**
```python
LORA_CONFIG = dict(
    r=16,
    lora_alpha=16,  # r=16: sufficient rank for narrow behavioral alignment —
                    # Aghajanyan et al. (2021): intrinsic dimensionality of
                    # preference fine-tuning is low; 381 pairs + 4 alignment
                    # dimensions → r=32 is ceiling; r=16 tests simpler cases.
    ...
)
```

---

## Why It Changed

Rahel's explainer revealed that rank controls how many dimensions of the weight-update space the adapter can span, and that behavioral alignment tasks are low-dimensional — suppressing banned phrases requires a directional shift in activation space, not new factual knowledge. Before the explainer, r=32 was a tutorial default; after it, the choice is justified by the intrinsic dimensionality argument (Aghajanyan et al., 2021) and the practical rule that 381 pairs with four alignment dimensions sits at the r=32 ceiling, making r=16 the right test for whether simpler single-dimension cases converge equally well at lower rank and lower parameter cost.
