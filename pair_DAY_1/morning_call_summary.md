# Call Summary — Day 1 (Evening Call, held afternoon)

**Date:** 2026-05-04  
**Topic:** Inference-time mechanics  
**Pair:** Chalie Lijalem + Addisu Taye

## Questions Before Sharpening

**Your draft question:**
> In my Week 10 Conversion Engine, the agent re-sends the same system prompt — Tenacious ICP definition, bench summary, and hiring signal brief — on every turn of the multi-turn loop. My `baseline.md` shows `avg_agent_cost: $0.0199` but some tasks cost up to $0.0963 (task_4). I claim the system prompt is "reused across turns" but I have never verified whether the LLM API is actually caching that repeated prefix or charging me full prefill cost every time. How does KV cache work at the API call boundary — what conditions must be met for prefix caching to apply, and how does cache invalidation happen when the conversation context changes between turns?

**Partner's draft question:**
> How is the total latency and cost of a single LLM inference call split between the prefill and decode phases, and how would changes in prompt length vs output length quantitatively shift both cost and p95 latency?

## Context

Pairing was assigned by the tutor team immediately after the topic announcement. The paired call happened in the evening of Day 1.

## Draft Questions Before the Call

**Chalie's draft:**
> My agent costs vary a lot across tasks. Why does my Conversion Engine show such high cost and latency variance in trace_log.jsonl?

**Addisu's draft:**
> What is the difference between the prefill and decode phases in LLM inference?

## What Was Interrogated and How Questions Moved

1. **Addisu pushed back on Chalie's draft:** "What specific mechanism are you pointing at — is it the number of API calls, the token count, or something about how the prompt is structured?" Chalie realized the question was aimed at a specific architectural choice: the system prompt is re-sent on every turn without knowing if the API caches it. The question sharpened from "why is cost high?" to "what conditions must be met for prefix caching to apply, and how does cache invalidation work?"

2. **Chalie pushed back on Addisu's draft:** "You're asking what the difference is — but that's a definitional question, answerable in a sentence. What would you need to know to actually make engineering decisions from it?" Addisu identified the real gap: not the names of the phases, but how changes in prompt length vs output length quantitatively shift both cost and p95 latency — two different dimensions that each phase dominates differently.

3. **Final direction confirmed:** Chalie's question anchors on the KV cache boundary and cache invalidation conditions — specific enough to produce a grounded explainer. Addisu's question anchors on the quantitative split — specific enough to resolve with worked examples from real trace data.

## Final Questions (committed after call)

**Chalie's final question:**
> In my Week 10 Conversion Engine, the agent re-sends the same system prompt — Tenacious ICP definition, bench summary, and hiring signal brief — on every turn of the multi-turn loop. My baseline.md shows avg_agent_cost: $0.0199 but some tasks cost up to $0.0963 (task_4). I claim the system prompt is "reused across turns" but I have never verified whether the LLM API is actually caching that repeated prefix or charging me full prefill cost every time. How does KV cache work at the API call boundary — what conditions must be met for prefix caching to apply, and how does cache invalidation happen when the conversation context changes between turns?

**Addisu's final question:**
> How is the total latency and cost of a single LLM inference call split between the prefill and decode phases, and how would changes in prompt length vs output length quantitatively shift both cost and p95 latency?

## Attestation
- [x] Addisu attests Chalie's question is unambiguous
- [x] Chalie attests Addisu's question is unambiguous
