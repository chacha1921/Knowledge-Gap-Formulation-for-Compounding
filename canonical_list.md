# Canonical List — Week 12 Cohort Contribution

**Contributor:** Chalie Lijalem
**Program:** TRP1 Forward Deployed Engineer — 10 Academy / Tenacious Intelligence Corp

Papers, tools, and patterns worth every Forward-Deployed Engineer reading. Each entry is annotated with why it matters for FDE work specifically — not just what it says.

---

## Papers

**Aghajanyan, Gupta, Zettlemoyer et al. — "Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning," 2021**
arXiv:2012.13255

Why it matters for FDE work: This is the justification for why LoRA rank selection is not arbitrary. The paper measures the intrinsic dimensionality of fine-tuning tasks and finds that most tasks — including behavioral alignment tasks like suppressing banned phrases — have intrinsic dimensionality in the hundreds, not millions. When a client asks "why r=32 and not r=64?", this paper is the answer. It also tells you when to go lower (single simple rules, small dataset → r=8–16) and when r=32 is the correct ceiling rather than a tutorial default.

---

**Hu, Shen, Wallis et al. — "LoRA: Low-Rank Adaptation of Large Language Models," 2021**
arXiv:2106.09685

Why it matters for FDE work: The foundational reference for any PEFT deployment. Section 2 derives the ΔW = BA decomposition and the α/r scaling. Section 4 contains the empirical result that r=4–8 is sufficient for many NLP tasks. FDE engineers need to understand that r controls how many dimensions the adapter can span and α/r controls how strongly the adapter influences each forward pass — these are separate knobs that interact differently with dataset size and task complexity.

---

**Yu, Zhu, Wang et al. — "SimPO: Simple Preference Optimization with a Reference-Free Reward," 2024**
arXiv:2405.14734

Why it matters for FDE work: If you are doing preference optimization without a reference model, SimPO is the current practical choice. Section 3 defines the γ margin term clearly and explains why it exists — DPO has no explicit margin; SimPO introduces γ to enforce minimum separation between chosen and rejected reward scores. The practical implication: γ must be calibrated to the discriminability of your preference pairs, not copied from the paper default. Weakly-annotated or domain-shifted data needs a lower γ.

---

**Rafailov, Sharma, Mitchell et al. — "Direct Preference Optimization: Your Language Model Is Secretly a Reward Model," NeurIPS 2023**
arXiv:2305.18290

Why it matters for FDE work: The reference for DPO, which is the baseline SimPO and ORPO improve on. Reading DPO and SimPO side by side shows exactly what the γ term adds and why reference-free methods trade KL regularization for explicit margin control. When a client asks why you chose SimPO over DPO, the mechanism in these two papers is the answer.

---

**Berg-Kirkpatrick, Burkett, & Klein — "An Empirical Investigation of Statistical Significance in NLP," EMNLP 2012**

Why it matters for FDE work: Directly addresses a failure mode FDE engineers encounter constantly: reporting "p < 0.05" or "not significant" from a paired bootstrap without understanding what the test is actually measuring. Section 3 shows that p-values from paired bootstrap depend on the number of discriminating items, not on n alone. When your evaluation has a high baseline (97%+), this paper explains why adding more pairs of the same type will not fix your CI — and what will.

---

**Dror, Baumer, Shlomov, & Reichart — "The Hitchhiker's Guide to Testing Statistical Significance in Natural Language Processing," ACL 2018**
arXiv:1809.01448

Why it matters for FDE work: A practical decision guide for choosing the right statistical test in NLP evaluation. Section 4 covers ceiling effects and the conditions under which bootstrap CIs are informative vs misleading. Table 2 is a reference table for when to switch from paired bootstrap to permutation tests or continuous scoring. Every FDE who has shipped a benchmark and needs to report results honestly should read Section 4 before writing the evaluation section of a model card.

---

**Li, Zhao, Li et al. — "Preference Leakage: A Contamination Problem in LLM-as-a-Judge," 2025**
arXiv:2502.01534

Why it matters for FDE work: Defines the taxonomy every FDE needs when designing judge-generator separation: same-model leakage, same-family leakage, and prompt-template leakage. Cross-family rotation (which most practitioners implement) closes same-model and same-family leakage but not prompt-template leakage. This paper is the reference for why your cross-family guard is necessary but not sufficient — and why you still need to test for self-preference separately.

---

**Zheng, Chiang, Sheng et al. — "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena," NeurIPS 2023**
arXiv:2306.05685

Why it matters for FDE work: Section 3.3 documents self-preference bias empirically: GPT-4 rates GPT-4 outputs higher than human judges do, and the gap is systematic across dimensions including tone and instruction-following. This is the canonical empirical reference for why cross-family rotation closes the leakage path but leaves the self-preference path open — and why Delta results are more trustworthy than absolute scores when the same judge family evaluates all conditions.

---

## Tools and Patterns

**`smart_bbox_crop()` — OpenCV VLM preprocessing**

```python
def smart_bbox_crop(img_bytes: bytes, padding: int = 20) -> bytes:
    nparr = np.frombuffer(img_bytes, np.uint8)
    img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    _, thresh = cv2.threshold(gray, 250, 255, cv2.THRESH_BINARY_INV)
    coords = cv2.findNonZero(thresh)
    if coords is None:
        return img_bytes
    x, y, w, h = cv2.boundingRect(coords)
    ih, iw = img.shape[:2]
    x, y = max(0, x - padding), max(0, y - padding)
    w, h = min(iw - x, w + 2 * padding), min(ih - y, h + 2 * padding)
    _, buf = cv2.imencode('.png', img[y:y+h, x:x+w])
    return buf.tobytes()
```

When to use: Before any VLM API call on document images. Crops to content bounding box, removing margins before tiling. Reduces tile count by ~40–50% on standard scanned documents. Centers tables relative to tile boundaries, reducing the probability that a row straddles a tile cut.

---

**`simpo_gradient_magnitude()` — SimPO training diagnostics**

```python
def simpo_gradient_magnitude(beta, margin, gamma):
    z = beta * margin - gamma
    sigmoid = 1 / (1 + math.exp(z))
    return beta * sigmoid
```

When to use: When diagnosing SimPO training instability. Given your β, the observed reward margin for a batch, and your γ, this returns the gradient magnitude. If the magnitude is large for most of your batch at epoch 1, γ is too high relative to your pair discriminability. Drop γ until the gradient is proportionate to the actual signal in your data.

---

**Binomial resampling decomposition pattern**

When a paired bootstrap returns a surprising CI, decompose it: with n pairs and k discriminating pairs, the CI upper bound equals (97.5th percentile of Binomial(n, k/n)) / n. If k=1, the upper bound reflects sampling noise around that single pair — not a range of plausible true effect sizes. This decomposition converts an opaque CI into a count of critical-pair occurrences and immediately tells you whether the interval is informative or a ceiling artifact.

---

**Three-component tool description pattern (for multi-tool agents)**

```
tool_name: One sentence — what this tool does.
Use when: [specific trigger condition with example].
NOT for: [specific anti-trigger — the most powerful component].
Prefer this over [competing tool] when [discriminating condition].
```

When to use: Any time two tools in your agent serve overlapping purposes. Anti-triggers ("NOT when X") suppress selection more reliably than trigger phrases. Test with 6–10 overlapping queries before deploying; any miss is a description bug, not a model limitation.
