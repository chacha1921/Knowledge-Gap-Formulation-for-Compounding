# Day 3 Gap Question

**Asker:** Chalie Lijalem
**Date:** 2026-05-08

---

During my Week 11 Sales-Agent-Evaluation-Bench ORPO fine-tuning, I set the LoRA adapter rank to r=32 and α=32. Mathematically, why is a low-rank matrix approximation capable of learning complex behavioral alignment (like avoiding banned phrases), and what fundamentally changes in the weight-update geometry if I scale it to r=64 or r=128?

Correspondingly, when computing the training loss, how does the Odds Ratio penalty in ORPO mathematically differ from the gradient mechanics of DPO or SimPO?
