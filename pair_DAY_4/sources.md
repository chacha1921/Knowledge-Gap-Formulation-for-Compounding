# Sources — Day 4

**Explainer written by:** Chalie Lijalem
**For:** Yonas Eshete

---

## Canonical Sources

**1. Berg-Kirkpatrick, Burkett, & Klein — "An Empirical Investigation of Statistical Significance in NLP," EMNLP 2012**

Load-bearing use: Section 3 demonstrates that paired bootstrap p-values in NLP evaluation are driven by the number of discriminating items, not by n alone. Directly underpins the claim that p=0.372 reflects the probability of the critical pair being absent from the resample rather than "insufficient sample size."

---

**2. Dror, Baumer, Shlomov, & Reichart — "The Hitchhiker's Guide to Testing Statistical Significance in Natural Language Processing," ACL 2018 (arXiv:1809.01448)**

Load-bearing use: Section 4 covers ceiling effects and the conditions under which bootstrap CIs are informative versus misleading. Table 2 provides guidance on when to switch from paired bootstrap to permutation tests or continuous scoring metrics — directly supports the recommendation to use reward margin rather than binary pairwise accuracy when the baseline is near-saturated.

---

## Tool / Pattern

**Binomial resampling analysis** — computing the exact distribution of the critical-pair appearance count under Binomial(44, 1/44) to verify that the 97.5th percentile at k=3 maps to exactly 6.8%, confirming the CI upper bound is a resampling artifact rather than a plausible effect size estimate. Implemented as a runnable Python script in the explainer's hands-on demonstration section.
