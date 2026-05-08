# Week 12 Synthesis — Knowledge Gap Formulation for Compounding

**Trainee:** Chalie Lijalem
**Program:** TRP1 Forward Deployed Engineer — 10 Academy / Tenacious Intelligence Corp
**Week:** 12 of 12

---

## What This Week Did

Weeks 1–11 built systems. Week 12 audited the understanding behind them. Each day I identified a genuine gap in my own prior work, sharpened the question with a partner, researched my partner's gap and wrote a public explainer, then committed a concrete improvement back to the system I had already shipped. The result is ten gaps closed — five I named, five I researched — and five real edits to portfolio work that now reflect the understanding the original code assumed but did not have.

---

## The Five Gaps I Named

**Day 1 — KV cache at the API call boundary**
My Week 10 Conversion Engine re-sent the same system prompt on every turn of its multi-turn loop. My `baseline.md` showed a 12x cost spread across tasks ($0.0077 to $0.0963) that I had attributed to task difficulty. The gap was that I did not understand whether the API was caching the repeated prefix or charging full prefill cost every turn. The explainer from Addisu revealed that KV cache does not persist across API call boundaries — each call rebuilds the cache from scratch — and that cost grows O(n²) in turns because each turn's input includes all prior context. The 12x spread is turn-count, not task difficulty.

**Day 2 — What `detail: high` actually does**
My Week 3 Document Intelligence Refinery escalated failed document pages to GPT-4o-mini with `detail: high`. I described it in my code comments as "escalates to vision model for complex layout." I could not predict when it would succeed or fail on tables, and I had no number for token cost per page. The explainer from Gashaw revealed the tiling mechanism: 512×512 crops, 170 tokens per tile plus an 85-token base overhead, a standard A4 at 150 DPI = 6 tiles = 1,105 tokens = $0.000166. More importantly, it revealed the structural failure mode: tiles are independent crops with no shared positional encoding across boundaries. A table row that straddles a tile boundary becomes two disconnected patches — the model cannot reconstruct the continuity. I can now predict failure from table width relative to tile size before making the API call.

**Day 3 — LoRA rank mechanics**
My Week 11 Sales-Agent-Evaluation-Bench ORPO fine-tuning used r=32 and α=32 because the Unsloth tutorial used those values. I could not explain why low-rank updates can learn behavioral alignment, what would change at r=8 or r=64, or why α and r were both set to 32. The explainer from Rahel grounded the choice in the intrinsic dimensionality argument (Aghajanyan et al., 2021): most fine-tuning tasks have intrinsic dimensionality in the hundreds, not millions. "Never say offshore" is a directional shift in activation space, not a factual rewrite. r=32 matches the task's natural dimensionality; r=64 risks overfitting on 381 pairs; α/r is a separate knob from r that controls update magnitude, not expressivity.

**Day 4 — Self-preference bias vs cross-family rotation**
My Week 11 tenacious-bench documented cross-family rotation as its evaluation integrity safeguard. I understood that this prevented preference leakage — same-family judge and generator — but I had never separated leakage from self-preference bias. The explainer from Yonas named the distinction: leakage is a data contamination problem (training lineage overlap), self-preference is an inference-time problem (judge's embedded aesthetic preferences operating at scoring time). Cross-family rotation closes the first path and does not touch the second. My pipeline had two live self-preference paths: Gemini evaluating tone on tasks it also authored, and Gemini generating chosen preference pairs and then judging the same tone dimension at held-out time.

**Day 5 — TBD**
*(To be completed.)*

---

## The Five Gaps I Researched

**Day 1 — Prefill/decode cost split (for Addisu Taye)**
Addisu's Conversion Engine showed cost variance he attributed to task difficulty. The gap was not understanding how a single LLM inference call is split into a prefill phase (reads all input tokens, quadratic cost in sequence length) and a decode phase (generates one token at a time, linear in output length). The load-bearing mechanism: prompt tokens dominate cost when the system prompt is long; output tokens dominate latency when generation is long. The practical implication for Addisu's multi-turn loop was that restructuring context management — truncating history rather than appending it indefinitely — would compress the most expensive phase without affecting output quality.

**Day 2 — Tool selection mechanism (for Gashaw Bekele)**
Gashaw's multi-tool agent was selecting the wrong tool when two tools served overlapping purposes. The gap was not understanding whether the LLM was reasoning over descriptions or the framework was doing routing. The answer: both, at different layers. The model uses next-token prediction over tool definitions in the context window to choose which tool; the framework constrains the output to be schema-conforming. ToolScope (Liu et al., 2025) quantified the overlap cost: 8.8–38.6% degradation in selection accuracy when descriptions are ambiguous. Anti-trigger phrases ("NOT when you have an exact ID") are consistently more powerful than trigger phrases — the model suppresses a tool more reliably than it selects one from a positive description.

**Day 3 — SimPO γ instability on weakly-discriminating pairs (for Rahel Samson)**
Rahel set γ=0.3 instead of the paper default γ=0.5 because γ=0.5 was unstable on her training data, but she could not explain the mechanism. The load-bearing mechanism: γ is the target reward margin — the minimum gap the model must open between chosen and rejected scores before the loss approaches zero. On weakly-discriminating pairs (chosen and rejected close in quality), the natural gap is small (~0.05–0.10). With γ=0.5, the required margin is far above what the data can support, producing disproportionately large gradient updates that force the model to make extreme adjustments damaging its base distribution. γ=0.3 is correct for Rahel's dataset because it is calibrated to the actual discriminability of her pairs.

**Day 4 — Paired bootstrap CIs at ceiling (for Yonas Eshete)**
Yonas's evaluation returned p=0.372 and CI=[0.0%, 6.8%] on 44 preference pairs with a 97.73% baseline. He attributed this to small n. The actual mechanism: with base=43/44 and adapter=44/44, only one pair discriminates between the two systems. P(that pair absent from a 44-draw resample) = (43/44)^44 ≈ 0.362, which is his p-value. The 6.8% upper bound is what the improvement looks like when the critical pair is drawn three times in the luckiest bootstrap resample — not a plausible effect size estimate but a resampling artifact from a 1/44 probability event at its 97.5th-percentile count. The ceiling effect does not violate bootstrap assumptions; it reveals the evaluation is structurally underpowered for the adversarial regime the adapter was trained on.

**Day 5 — TBD**
*(To be completed.)*

---

## The Most Surprising Thing I Learned

Self-preference inflation affects absolute scores but not relative deltas. I had assumed that if a judge family is evaluating its own preferred style, the entire result set is compromised. Yonas's explainer showed the opposite: baseline and adapter are both scored by the same judge under the same stylistic bias, so the bias cancels in the delta. Delta A remains the correct number even if absolute scores are inflated. This has a concrete implication for FDE work: the evaluation integrity question to ask is not "is there bias?" but "does the bias affect the comparison or only the absolute level?" — two different audits requiring two different tests.

The second surprise was the binomial result behind bootstrap CIs. I had treated confidence intervals as straightforward summaries of effect-size uncertainty. The one-pair bootstrap analysis showed that CI=[0.0%, 6.8%] is entirely determined by the sampling distribution of a single pair — the upper bound is k=3 occurrences of that pair at the 97.5th percentile of Binomial(44, 1/44). The interval communicates resampling noise, not a range of plausible true improvements. A CI can be mathematically correct and nearly uninformative at the same time.

---

## Canonical Reading List Contributed to Cohort Canon

See `canonical_list.md` for the annotated entries. The load-bearing papers this week:

- Aghajanyan et al. (2021) — intrinsic dimensionality argument that grounds LoRA rank selection
- Hu et al. (2021) — the LoRA paper itself; Section 4 on why r=4–8 suffices for many NLP tasks
- Yu et al. (2024) — SimPO; Section 3 defines γ and explains its role directly
- Berg-Kirkpatrick et al. (2012) — bootstrap behavior in NLP evaluation under low-discrimination conditions
- Dror et al. (2018) — the reference for when to switch from bootstrap to permutation tests or continuous scoring
- Li et al. (2025) — preference leakage taxonomy; distinguishes same-model, same-family, and prompt-template leakage
- Zheng et al. (2023) — empirical documentation of self-preference bias; Section 3.3 is the FDE-grade reference

## Tool List Contributed to Cohort Canon

- `smart_bbox_crop()` — 15-line OpenCV preprocessing; crops page to content bounding box before VLM tiling, reducing tile count by ~40–50% and centering tables relative to tile boundaries
- `simpo_gradient_magnitude()` — Python script; given β, margin, and γ, returns gradient magnitude; makes the instability mechanism visible and verifiable
- Binomial resampling decomposition — bootstrap CI upper bound = k-th percentile of Binomial(n, 1/n); converts an opaque CI into a count of critical-pair occurrences
- Generation-model stratification script — stratifies held-out scores by `generation_model` field; quantifies self-preference inflation without additional API calls
