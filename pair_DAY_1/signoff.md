# Gap Sign-off — Day 1

**Asker:** Addisu Taye  
**Explainer written by:** Chalie Lijalem  
**Question:** How is the total latency and cost of a single LLM inference call split between the prefill and decode phases, and how would changes in prompt length vs output length quantitatively shift both cost and p95 latency?

---

## Addisu's Sign-off on Chalie's Blog (prefill/decode question)

**Gap status:**
- [x] Fully closed
- [ ] Partially closed
- [ ] Not closed

**What landed:**

Before this explainer, Addisu believed the system prompt was effectively "reused" across turns and therefore not a major contributor to cost. The explainer clarified that while the prompt is logically reused, it is computationally recomputed on every API call because KV cache does not persist across API boundaries.

Addisu also learned that prefix caching does not reliably apply: it requires a byte-identical prefix starting from token position 0, stable routing, and short cache lifetimes — conditions rarely satisfied in a multi-turn agent where conversation history grows dynamically.

The cost variance is now precisely attributable: tasks with longer trajectories (task_4 at $0.0963) accumulate more context and incur repeated prefill computation, while shorter trajectories (task_73 at $0.0077) remain cheap. The difference is token growth across turns, not task difficulty.

**Signed off by:** Addisu Taye — 2026-05-05 11:48 PM

---

## Chalie's Sign-off on Addisu's Explainer (KV cache question)

**Gap status:**
- [x] Fully closed
- [ ] Partially closed
- [ ] Not closed

**What the explainer revealed that I did not know before:**

Three things I now understand that I did not before:

1. **KV cache is not preserved across API calls.** I had assumed "conversational state is maintained across turns" meant the model was reusing its internal representation. It is not. Each API call rebuilds the KV cache from scratch starting from the full input — system prompt, history, everything.

2. **The cost growth is O(n²) in turns.** Each turn appends prior context to the next input. By turn 3, you are paying to process all of turn 1 and turn 2 again. My highest-cost tasks (task_4 at $0.0963) ran more tool-call turns — this is the mechanism.

3. **Prefix caching is real but narrow.** It only applies when the prefix is byte-identical, hits the same infrastructure, and the cache hasn't expired. In a multi-turn agent where the conversation history grows each turn, the shared prefix (the system prompt) becomes a shrinking fraction of total tokens. Caching saves diminishing amounts as context grows.

**What the explainer missed (raised in evening call):**
- No canonical papers cited — the rubric requires 2
- Verification section has pseudocode only — no code actually run, no output shown
- Prefix caching conditions incomplete — the most critical condition (prefix must be byte-identical from position 0) was not named explicitly

**Grounding commit made:**
> See `grounding_commit.md`
