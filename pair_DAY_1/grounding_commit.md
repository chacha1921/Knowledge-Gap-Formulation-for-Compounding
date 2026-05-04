# Grounding Commit — Day 1

**Gap closed:** KV cache mechanics — how prefix caching works at the API call boundary, why costs grow O(n²) across turns, and what drives the p95 latency spike

---

## What I Now Understand (from Addisu's explainer)

Three things changed in how I read my own traces:

1. **KV cache is not preserved across API calls.** My agent re-sends the full system prompt + conversation history on every turn. I was not paying once for a reused representation — I was re-paying prefill cost on the full accumulated context every single turn.

2. **The cost growth is O(n²) in turns.** By turn 3, the input includes all of turns 1 and 2. High-cost tasks like task_4 ($0.0963) and task_105 ($0.0998) ran more tool-call turns, not harder tasks — the mechanism is context accumulation, not task difficulty.

3. **p95 latency is not all inference.** Some traces show high duration but low cost (task_92: 687s, $0.0124). The agent was waiting for the environment, not generating tokens. My p95 of 551s conflates inference time with environment wait time — these are different problems requiring different fixes.

---

## Concrete Edit Made

**File edited:** `week10/Technical Challenge/baseline.md`

**What changed:** Added a "Cost Interpretation" section that:
- Explains the 14.7x cost spread (not just the average)
- Names the O(n²) context growth mechanism
- Notes that KV cache is not preserved across API calls
- Explains why p95 latency conflates inference and environment wait time
- Documents the prefix caching conditions and why they give diminishing returns as context grows

**Before:** baseline.md reported `avg agent cost: $0.0199` with no explanation of variance or underlying mechanism.

**After:** baseline.md now documents the mechanism behind the cost distribution, making it defensible to a hiring manager or technical reviewer.

---

## Gap Sign-off
- [x] Gap closed
- [ ] Partially closed
- [ ] Not closed
