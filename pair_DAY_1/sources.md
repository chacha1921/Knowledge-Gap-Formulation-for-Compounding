# Sources — Day 1

**Explainer written by:** Chalie Lijalem  
**Question answered:** Addisu Taye's question on prefill vs decode cost and latency split

---

## Primary Sources Read

### 1. PagedAttention — vLLM (load-bearing)
**Citation:** Kwon et al., *Efficient Memory Management for Large Language Model Serving with PagedAttention*, SOSP 2023  
**Why canonical:** This is the foundational systems paper for modern LLM serving. It explicitly defines and schedules prefill and decode as distinct phases — Section 3 describes the KV cache lifecycle across both phases in detail. The entire PagedAttention memory management design follows from KV cache being created during prefill and read sequentially during decode.  
**How it is load-bearing:** The explanation of why decode is memory-bandwidth-bound (the KV cache must be read from GPU memory on every token step) comes directly from this paper's analysis.

### 2. Efficiently Scaling Transformer Inference (load-bearing)
**Citation:** Pope et al., *Efficiently Scaling Transformer Inference*, MLSys 2023  
**Why canonical:** Google Research's analysis of transformer inference at production scale. Section 4 presents the hardware roofline model showing prefill as compute-bound and decode as memory-bandwidth-bound — the core claim of this explainer.  
**How it is load-bearing:** The claim "Modern GPUs have far more compute (FLOPS) than memory bandwidth (GB/s)" and the explanation of why the two phases hit different hardware ceilings comes from this paper's roofline analysis.

---

## Tool Used

**trace_log.jsonl** — 150 inference traces from the Week 10 Conversion Engine (30 tasks × 5 trials). Analysis script in `explainer_blog.md` is directly runnable against this file. Shows: same-task cost variance (4.2x–7.1x), cost-duration decoupling (agent waiting vs generating), and quantitative cost breakdown by input/output token count.

---

## What I Did Not Use

- Second-hand blog summaries of these papers (propagate errors, obscure mechanisms)
- The Attention Is All You Need paper (Vaswani et al., 2017) — foundational for transformer architecture but does not discuss inference-time phase distinction
- Speculative decoding paper (Leviathan et al., ICML 2023) — relevant adjacent concept but not load-bearing for the central question; referenced briefly in connecting-the-dots section only
