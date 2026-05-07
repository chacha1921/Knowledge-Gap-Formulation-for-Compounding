# Tweet Thread — Day 3

**Topic:** What γ controls in SimPO and why it breaks on weak preference pairs
**Author:** Chalie Lijalem
**Date:** 2026-05-08
**Medium:** https://medium.com/@chalielijalem/explainer-what-%CE%B3-controls-in-simpo-and-why-it-breaks-on-weak-preference-pairs-67b83eab169f
**Twitter:** https://x.com/ChalieLijalem/status/2052382720240583059?s=20

---

**Tweet 1**
SimPO has a hyperparameter γ most people just copy from the paper. My peer changed it — and her training was more stable. Here's why that was the right call 🧵

---

**Tweet 2**
γ is the margin. It sets the minimum gap the model must open between the chosen and rejected response scores. Not just "chosen > rejected" — chosen must be higher BY AT LEAST γ.

---

**Tweet 3**
When chosen and rejected responses are close in quality, the natural gap is tiny. A high γ demands a gap the data can't support. Gradients get huge. Training breaks.

---

**Tweet 4**
Lower γ = smaller required gap = stable gradients. Match γ to your data's signal strength, not the paper default tuned on clean, well-separated pairs.

---

**Tweet 5**
Always know WHY your hyperparameter works — not just that it does. Sources: SimPO (arXiv:2405.14734) · DPO (arXiv:2305.18290)
