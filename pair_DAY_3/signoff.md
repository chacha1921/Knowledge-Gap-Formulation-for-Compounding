# Sign-off — Day 3

**Asker:** Chalie Lijalem
**Explainer written by:** Rahel Samson
**Gap-closure judgment:** Closed

---

Before this explainer I had set r=32 and α=32 because the Unsloth tutorial used those values, with no understanding of what rank controls or what would change at r=8 or r=64. I now understand that low-rank updates work because behavioral alignment tasks have low intrinsic dimensionality — suppressing banned phrases is a directional shift in activation space, not a factual rewrite, and that shift lives in a small subspace that r=32 captures. I understand that r=8 is too narrow for multi-rule alignment, r=64 risks overfitting on my 381-pair dataset, and r=32 is the correct ceiling for my task. I also now understand that α/r is a separate knob from r — r controls how many dimensions the adapter spans, α/r controls how strongly the adapter influences each forward pass — and that setting both to 32 was a coincidence, not a principled choice.
