# Tweet Thread — Day 4

**Author:** Chalie Lijalem
**Topic:** What p=0.372 actually says when your baseline is 97.73%
**Published:** https://x.com/ChalieLijalem/status/2052753337041223996?s=20
**Medium:** https://medium.com/@chalielijalem/explainer-what-p-0-372-actually-says-when-your-baseline-is-97-73-58b4fcdc1668

---

**Tweet 1** (158 chars)
Your bootstrap test returns p=0.372, CI=[0.0%, 6.8%]. Base model: 97.73%. Adapter: 100%. You call it "not significant, small n." That's not the diagnosis. Here's what's actually happening.

---

**Tweet 2** (196 chars)
With 43/44 base and 44/44 adapter, only ONE pair discriminates between the two systems. P(that pair missing from any bootstrap resample) = (43/44)^44 ≈ 0.362. That's your p-value. Not small n. One-pair problem.

---

**Tweet 3** (212 chars)
The CI=[0.0%, 6.8%] maps to k=0,1,2,3 occurrences of that pair in the resample. The 6.8% upper bound is what the improvement looks like when the pair is drawn 3x by luck — a resampling artifact, not a plausible effect size.

---

**Tweet 4** (188 chars)
The ceiling effect doesn't break bootstrap assumptions. It reveals the evaluation can't answer the question. 76/100 dev pairs were silent passengers — near-zero gradient, already saturated. The adversarial probes were never evaluated.

---

**Tweet 5** (176 chars)
Three fixes, ranked by cost: (1) evaluate on the adversarial slice directly, (2) switch to generation-mode scoring — harder to saturate than pairwise log-prob, (3) use continuous reward margin instead of binary win/lose.

---

**Tweet 6** (144 chars)
More silent-passenger pairs doesn't fix this. A 10-pair adversarial slice at 65% base accuracy is more informative than 440 pairs at 97.73%.
