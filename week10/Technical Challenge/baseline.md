# τ² retail baseline

## Summary
- evaluated simulations: 150
- infra errors: 0
- total tasks: 30
- pass@1: 0.7267
- 95% CI: [0.6504, 0.7917]
- avg agent cost: $0.0199
- p50 latency: 105.9521s
- p95 latency: 551.6491s

## Provenance
- git commit: d11a97072c49d093f7b5a3e4fe9da95b490d43ba
- trials per task: 5

## Cost Interpretation (added Week 12 Day 1)

`avg agent cost: $0.0199` is an average across 150 runs (30 tasks × 5 trials). It does not reflect the actual cost structure, which has a 14.7x spread (min $0.0068, max $0.0998).

**Why costs vary so much:**
- Each inference call has two phases: prefill (input tokens, processed in parallel) and decode (output tokens, generated one at a time)
- In a multi-turn agent loop, each turn appends prior conversation to the input — context grows O(n²) across turns
- The KV cache is NOT preserved across API calls; every turn re-pays full prefill cost on the accumulated context
- High-cost tasks (task_4: $0.0963, task_105: $0.0998) ran more tool-call turns, compounding both prefill and decode costs

**Why p95 latency (551s) is not purely inference:**
- Some high-duration traces show cost-duration decoupling: task_92 spent 687s at only $0.0124
- This indicates the agent was waiting for an external action (user_stop), not generating tokens
- p95 latency conflates inference time with environment wait time

**Prefix caching note:**
- The agent re-sends the same system prompt (Tenacious ICP, bench summary, hiring signal brief) on every turn
- Prefix caching at the API level would reduce this cost but only applies when: (1) the prefix is byte-identical from position 0, (2) requests hit the same infrastructure, (3) cache has not expired
- As conversation history grows each turn, the cacheable prefix (the static system prompt) becomes a shrinking fraction of total tokens — caching savings diminish with turn count
