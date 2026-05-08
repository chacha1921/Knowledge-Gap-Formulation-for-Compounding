# Tweet Thread — Day 1

**Question answered:** How is the cost and latency of a single LLM inference call split between prefill and decode phases?

**Published URL:** https://x.com/ChalieLijalem/status/2051408252194193704?s=20

---

**Tweet 1 — The question**
> Two tasks in my agent trace: one costs $0.007 and takes 43s. Another costs $0.096 and takes 683s. Same model. Same agent. 12x cost difference.
>
> The answer lives inside how every LLM inference call splits into two fundamentally different phases: prefill and decode. 🧵

---

**Tweet 2 — Prefill**
> Phase 1: Prefill
>
> All your input tokens — system prompt, history, user message — are processed IN PARALLEL in one forward pass.
>
> Output: the KV cache (stored attention keys + values for every input token).
> Bottleneck: compute (matrix multiplication).
> Latency metric: Time to First Token (TTFT).

---

**Tweet 3 — Decode**
> Phase 2: Decode
>
> Output tokens are generated ONE AT A TIME. Each token requires reading the full KV cache built in prefill.
>
> Cannot be parallelized — it's sequential by design.
> Bottleneck: memory bandwidth (reading the KV cache).
> Latency metric: TPOT × number of output tokens.

---

**Tweet 4 — The numbers**
> The cost asymmetry is real. On Claude Sonnet 4.6:
>
> Input tokens (prefill): $3/1M
> Output tokens (decode): $15/1M → 5x more expensive
>
> Double your prompt length → doubles prefill cost, raises TTFT
> Double your output length → doubles decode cost AND dominates p95 latency
>
> Output length is the p95 killer. Prompt length is the cost creep.

---

**Tweet 5 — The adjacent insight**
> This is why speculative decoding speeds up inference.
>
> Prefill is already parallel — nothing to optimize there.
> Decode is the bottleneck: sequential, memory-bound, one token at a time.
>
> Speculative decoding uses a small draft model to propose tokens in parallel, then verifies them in a single prefill-like pass. It's attacking exactly the right phase.

---

**Tweet 6 — Link**
> Full explainer with worked cost calculations, a real agent trace breakdown, and pointers to the vLLM (PagedAttention) and speculative decoding papers:
>
> https://chalielijalem.substack.com/p/the-two-phases-inside-every-llm-call
>
> Written as part of @10Academy TRP1 Week 12 — closing inference-time mechanics gaps grounded in real agent traces.

---

*Each tweet stands alone without clicking through.*
