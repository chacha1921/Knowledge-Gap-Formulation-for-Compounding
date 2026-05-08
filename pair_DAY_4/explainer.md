# Explainer — What p=0.372 Actually Says When Your Baseline Is 97.73%

**Written by:** Chalie Lijalem
**For:** Yonas Eshete
**Date:** 2026-05-08
**Published:** https://medium.com/@chalielijalem/explainer-what-p-0-372-actually-says-when-your-baseline-is-97-73-58b4fcdc1668

---

## The Question

Yonas ran a paired bootstrap test on 44 held-out preference pairs. The base model scored 43/44 (97.73%); the SimPO LoRA adapter scored 44/44 (100%). The bootstrap returned p=0.372 and 95% CI=[0.0%, 6.8%]. He attributed non-significance to small n and a ceiling effect, but could not defend whether the CI is interpretable at this baseline or what it would actually take to reach a trustworthy conclusion.

---

## One Discriminating Pair Is Doing All the Work

The paired bootstrap resamples your 44 pairs with replacement and computes the difference in accuracy across 1000 iterations. With base=43/44 and adapter=44/44, only one pair discriminates between the two systems — call it the critical pair. In every bootstrap sample, the adapter scores 44 correct because it got all 44 original pairs right. The base model's score varies only based on whether the critical pair lands in the resample.

The probability that the critical pair is absent from any single bootstrap draw of 44 items is:

```
P(absent) = (43/44)^44 ≈ 0.362
```

When the critical pair is absent, the difference is 0.0%. When it appears once, the difference is 1/44 = 2.27%. When it appears twice, 4.54%. When it appears three times, 6.82%. The p-value of 0.372 is approximately this absence probability — the fraction of bootstrap iterations where the apparent improvement was zero or less. This is not a small-n problem. It is a one-pair problem.

---

## What the CI=[0.0%, 6.8%] Actually Measures

The CI bounds map directly to the count of times the critical pair appears in the resample. The count follows a Binomial(44, 1/44) distribution — mean=1, and the 97.5th percentile falls at k=3 occurrences:

```python
import numpy as np
from scipy import stats

np.random.seed(42)
n = 44
base = np.ones(n); base[0] = 0   # wrong on pair 0 only
adapter = np.ones(n)              # correct on all

diffs = []
for _ in range(1000):
    idx = np.random.choice(n, size=n, replace=True)
    diffs.append(adapter[idx].mean() - base[idx].mean())

diffs = np.array(diffs)
print(f"p-value:  {(diffs <= 0).mean():.3f}")
print(f"95% CI:   [{np.percentile(diffs, 2.5):.4f}, {np.percentile(diffs, 97.5):.4f}]")
print(f"CI as pp: [{np.percentile(diffs, 2.5)*100:.1f}%, {np.percentile(diffs, 97.5)*100:.1f}%]")
```

**Output:**
```
p-value:  0.372
95% CI:   [0.0000, 0.0682]
CI as pp: [0.0%, 6.8%]
```

The 6.8% upper bound is not saying the adapter could be 6.8% better in reality. It is saying that in the luckiest bootstrap resample — where the critical pair is drawn three times — the apparent improvement looks like 6.8%. That is a resampling artifact from a 1/44 probability event appearing in its 97.5th-percentile count. The CI is mathematically correct. It is also nearly uninformative about the true effect size.

---

## Does the Ceiling Effect Violate Bootstrap Assumptions?

No — but it reveals why the evaluation is structurally unable to answer the question. The paired bootstrap assumes: (1) pairs are exchangeable (IID), and (2) the test statistic is a consistent estimator of the true difference. Neither assumption is violated. The problem is that with 43/44 pairs returning the same answer for both systems, the bootstrap has almost no signal to work with. The interval correctly quantifies sampling uncertainty around that one pair — it just so happens that one pair cannot carry the inference you need.

The diagnostic makes this structural problem concrete: 76 of 100 dev pairs were `silent_passenger` (margin ≥ 1.75, near-zero gradient contribution). The two pairs the adapter actually flipped were non-adversarial easy edges. The adversarial probes (P007/P011/P027) that drove the Path B decision were never in the evaluation slice. You built a bench to detect adversarial-probe improvement and then evaluated on a slice where the base model was already saturated on non-adversarial cases.

---

## The Honest Statement for the Model Card

Replace lines 63–67 with:

> The paired bootstrap returned p=0.372, 95% CI=[0.0%, 6.8%] (n=44, 1000 iterations, seed=42). This result should not be read as "the improvement is likely somewhere between 0% and 6.8%." With a 97.73% baseline, only one pair discriminated between the adapter and the base model; the CI reflects the sampling variability of that single pair, not a range of plausible true effect sizes. The evaluation cannot confirm or rule out a real improvement — it is structurally underpowered for the regime the adapter was trained on.

---

## What Would Actually Fix It

Three paths, in order of implementation cost:

**1. Evaluate on the adversarial slice directly.** The P007/P011/P027 probes drove the Path B decision. A 10–15 pair adversarial slice where the base model scores 60–70% is far more informative than 44 pairs at 97.73%.

**2. Generation-mode scoring.** Pairwise log-prob saturates because the base model already assigns higher log-prob to the correct response on 43/44 pairs. Generation-mode scoring is continuous rather than binary — harder to saturate at any baseline.

**3. Continuous reward margin.** Record r_chosen − r_rejected per pair instead of win/lose. The adapter might push a 0.05 margin to 0.25 — a real signal that binary accuracy collapses to "both correct."

Adding more non-adversarial pairs does not fix this. Switching to a permutation test or Fisher's exact test does not fix this either — a different test statistic on the same one-discriminating-pair data gives you the same answer. The problem is the data structure, not the test.

---

## Canonical Sources

1. **Berg-Kirkpatrick, Burkett, & Klein, "An Empirical Investigation of Statistical Significance in NLP," EMNLP 2012** — directly addresses paired bootstrap behavior in NLP evaluation, including under low-discrimination conditions. Section 3 shows that p-values from paired bootstrap depend heavily on the number of discriminating items rather than n alone.

2. **Dror, Baumer, Shlomov, & Reichart, "The Hitchhiker's Guide to Testing Statistical Significance in Natural Language Processing," ACL 2018** (arXiv:1809.01448) — Section 4 covers ceiling effects and the conditions under which bootstrap CIs are informative. Table 2 gives guidance on when to switch from paired bootstrap to permutation tests or continuous scoring.
