# Day 1 Cohort Presentation — Q5
**Presenter:** Chalie Lijalem | **Pair:** Addisu Taye | **Date:** 2026-05-06

---

## Slide 1 — The Gap: What I Claimed Without Verifying

**The artifact:** Week 10 Conversion Engine — 150 traces, 30 tasks × 5 trials

**The anomaly in my own data:**

| | task_73 | task_4 |
|---|---|---|
| Cost | $0.0077 | $0.0963 |
| Duration | 43s | 683s |
| Cost ratio | — | **14.7× more expensive** |

Same model. Same agent. Same reward outcome.

**What I wrote in baseline.md:**
> avg_agent_cost: $0.0199

One number. No mechanism. No explanation of the variance.

**The claim I made but never verified:**
> "The system prompt is reused across turns."

I wrote this assuming the KV cache would handle the repeated prefix. I had never verified whether the API was actually caching that prefix or charging me full prefill cost on every turn.

**The question I asked Addisu:**
> How does KV cache work at the API call boundary — what conditions must be met for prefix caching to apply, and how does cache invalidation happen when the conversation context changes between turns?

---

## Slide 2 — What the Explainer Revealed

**Partner:** Addisu Taye

Three things changed in how I read my own traces:

### Finding 1 — KV cache is NOT preserved across API calls
- Every turn rebuilds the KV cache from scratch
- The clearest sentence in Addisu's explainer: *"logically reused, computationally recomputed every time"*
- My agent was re-paying full prefill cost on the accumulated context on every single turn

### Finding 2 — The cost growth is O(n²) in turns
- Turn 1 input: system prompt + message 1
- Turn 3 input: system prompt + all prior messages + all prior responses + message 3
- Context grows with every turn — so prefill cost grows with every turn
- task_4 ($0.0963, 683s) ran more tool-call turns than task_73 ($0.0077, 43s)
- **It was not a harder task. It was more turns.**

### Finding 3 — Prefix caching has 5 strict conditions, all must be met simultaneously

Addisu documented 5 conditions — all required, any failure kills the cache hit:

| Condition | Why it breaks in my system |
|-----------|---------------------------|
| Byte-identical prefix from position 0 | Any dynamic metadata before the system prompt breaks it entirely |
| Exact token match — no whitespace differences | Formatting changes invalidate the cache |
| Same infrastructure path | No guarantee across API calls |
| Cache TTL still valid | Expires in seconds to minutes |
| Provider must support it | Not guaranteed across all APIs |

The most critical: **matching must start at token 0**. If any dynamic content (timestamps, session IDs, metadata) appears before the system prompt, prefix caching never applies — even if the system prompt itself is unchanged.

As conversation history grows, the reusable prefix shrinks relative to total input. Prefix caching gives **diminishing returns** as turns increase — the opposite of what I assumed.

### Canonical sources Addisu cited in the explainer:

| Source | What it answered |
|--------|-----------------|
| Kwon et al., *PagedAttention* — SOSP 2023 | KV cache structure and lifecycle — created during prefill, reused during decode, discarded after each API call |
| Anthropic, *Prompt Caching Documentation* | All 5 prefix caching conditions; why multi-turn agents cannot rely on it |

*(Pope et al., MLSys 2023 appeared in Addisu's sources.md as background reading but was not formally cited in the explainer.)*

### Hands-on experiment Addisu ran:

```python
start_time = time.time()
response = client.chat.completions.create(model="gpt-4o-mini",
    messages=[{"role": "user", "content": prompt}])
print("Prompt tokens:", response.usage.prompt_tokens)   # → 18
print("Completion tokens:", response.usage.completion_tokens)  # → 22
print("Total latency:", round(time.time() - start_time, 3), "seconds")  # → 0.82s
```

Running the same prompt twice produced nearly identical latency — no observable prefix caching. As prompt size increased across simulated turns, `prompt_tokens` and latency both grew proportionally — confirming prefill is recomputed, not reused.

---

## Slide 3 — What Changed in My Existing Work

**File edited:** `week10/Technical Challenge/baseline.md`

**Before** — one number, no mechanism:
```
avg_agent_cost:  $0.0199
p95_latency:     551.65s
```

Reported as if these were simple summaries. They are not.

**After** — added a Cost Interpretation section that explains:

1. The 14.7× cost spread (task_4 vs task_73) is caused by O(n²) context growth across turns, not task difficulty
2. KV cache is not preserved across API calls — the system prompt is re-prefilled on every turn
3. p95 latency (551s) conflates inference time with environment wait time — task_92 spent 687 seconds at only $0.0124 because the agent was waiting, not generating
4. Prefix caching conditions and why they give diminishing returns as context grows

**The baseline.md is now defensible** — it documents the mechanism behind the distribution, not just the average.

---

## One-sentence summary

> I wrote "the system prompt is reused across turns" — but every API call rebuilds the KV cache from scratch, context grows O(n²), and the 14.7× cost spread in my traces is explained entirely by turn count, not task difficulty.
