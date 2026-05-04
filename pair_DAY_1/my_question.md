# My Question — Day 1

**Subtopic:** KV cache mechanics; why prefix caching matters for cost; how cache invalidation actually works.

**Artifact referenced:** `week10/Technical Challenge/trace_log.jsonl`, `week10/Technical Challenge/baseline.md`

## Final Question

> In my Week 10 Conversion Engine, the agent re-sends the same system prompt — Tenacious ICP definition, bench summary, and hiring signal brief — on every turn of the multi-turn loop. My `baseline.md` shows `avg_agent_cost: $0.0199` but some tasks cost up to $0.0963 (task_4). I claim the system prompt is "reused across turns" but I have never verified whether the LLM API is actually caching that repeated prefix or charging me full prefill cost every time. How does KV cache work at the API call boundary — what conditions must be met for prefix caching to apply, and how does cache invalidation happen when the conversation context changes between turns?

## Connection to Existing Work

- **File:** `week10/Technical Challenge/trace_log.jsonl`
- **Specific anomaly:** task_4 costs $0.0963 (683s) vs task_73 costs $0.0077 (43s) — 12x cost difference
- **Language I used without understanding:** "the agent maintains conversational state across turns"
- **What would change if gap is closed:** Can instrument traces to verify prefix cache hits, and restructure system prompt to maximize caching benefit

## Gap Triage

- [x] Non-trivial (not a lookup)
- [x] Connected to a specific existing artifact
- [x] Closure would change something concrete in my work
- [x] Has a satisfying answer
