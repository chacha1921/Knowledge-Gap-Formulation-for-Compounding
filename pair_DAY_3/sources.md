# Sources — Day 3 Explainer

**Explainer written by:** Chalie Lijalem
**For question by:** Rahel Samson
**Date:** 2026-05-08

---

1. **Yu et al., "SimPO: Simple Preference Optimization with a Reference-Free Reward," 2024**
   arXiv:2405.14734
   Primary source. Defines the SimPO loss function, the γ margin term, and the length-normalized reward. Section 3 explains the role of γ directly.

2. **Rafailov et al., "Direct Preference Optimization: Your Language Model is Secretly a Reward Model," NeurIPS 2023**
   arXiv:2305.18290
   Baseline comparison. DPO has no explicit margin term. Comparing the two loss functions shows exactly what γ adds in SimPO — a minimum required separation between chosen and rejected reward scores that DPO does not enforce.

---

## Tool / Pattern Used

**Contrastive loss analysis pattern** — explaining a hyperparameter by computing the loss value at two different settings (γ=0.5 vs γ=0.3) on the same example gap (0.1), then showing how the sign and magnitude of the result changes. This makes an abstract margin term concrete without requiring calculus.
