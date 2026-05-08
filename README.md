# TRP1 Week 12 — Knowledge Gap Formulation for Compounding

**Program:** TRP1 Forward Deployed Engineer — 10 Academy / Tenacious Intelligence Corp  
**Trainee:** Chalie Lijalem  
**Week:** 12 of 12  
**Topic:** Knowledge Gap Formulation for Compounding

---

## What This Week Is

Weeks 1–11 built systems. Week 12 audits the understanding behind them. Each day:
- Identify one genuine knowledge gap from your own prior work
- Sharpen the question with a randomly paired partner
- Research your partner's question; write a public explainer
- Commit a concrete improvement back to your Week 10/11 portfolio

---

## Daily Topics

| Day | Topic | My Question Subtopic |
|-----|-------|----------------------|
| 1 | Inference-time mechanics | KV cache mechanics; why prefix caching matters for cost; how cache invalidation actually works |
| 2 | Function-calling agents | Tool selection and routing mechanics in multi-tool agents |
| 3 | Training and post-training mechanics | LoRA rank mechanics — why r=32 suffices for behavioral alignment and what changes at r=64 |
| 4 | Evaluation and statistics | Self-preference bias vs cross-family rotation — whether the leakage guard also closes the stylistic scoring path |
| 5 | TBD | |

---

## Public Artifacts

| Day | Blog Post | Tweet Thread |
|-----|-----------|--------------|
| 1 | [The Two Phases Inside Every LLM Call](https://medium.com/@chalielijalem/the-two-phases-inside-every-llm-call-prefill-decode-and-why-the-split-changes-everything-d35946f5aa4f) | [Tweet Thread](https://x.com/ChalieLijalem/status/2051408252194193704?s=20) |
| 2 | [How Tool Selection Actually Works in Function-Calling Agents](https://medium.com/@chalielijalem/how-tool-selection-actually-works-in-function-calling-agents-1961c1d3d222) | [Tweet Thread](https://x.com/ChalieLijalem/status/2052091935687393364?s=20) |
| 3 | [What γ Controls in SimPO and Why It Breaks on Weak Preference Pairs](https://medium.com/@chalielijalem/explainer-what-%CE%B3-controls-in-simpo-and-why-it-breaks-on-weak-preference-pairs-67b83eab169f) | [Tweet Thread](https://x.com/ChalieLijalem/status/2052382720240583059?s=20) |
| 4 | [What p=0.372 Actually Says When Your Baseline Is 97.73%](https://medium.com/@chalielijalem/explainer-what-p-0-372-actually-says-when-your-baseline-is-97-73-58b4fcdc1668) | [Tweet Thread](https://x.com/ChalieLijalem/status/2052753337041223996?s=20) |
| 5 | | |

---

## Portfolio Grounding (Week 10 & 11 edits)

See [portfolio_update.md](portfolio_update.md) for the full summary.

| Day | File Edited | What Changed |
|-----|-------------|--------------|
| 1 | `week10/Technical Challenge/baseline.md` | Added KV cache cost analysis and prefix caching strategy |
| 2 | `week11/agent_config.md` | Updated tool routing logic based on function-calling selection mechanism |
| 3 | `training/train_orpo.py` (Sales-Agent-Evaluation-Bench) | Reduced LoRA rank r=32 → r=16; justified by intrinsic dimensionality argument (Aghajanyan et al. 2021) |
| 4 | `tenacious-bench/methodology.md` | Added self-preference bias section with pending cross-judge agreement audit; qualified Delta A in model_card.md |
| 5 | | |

---
