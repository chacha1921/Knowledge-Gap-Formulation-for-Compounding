# The Two Phases Inside Every LLM Call: Prefill, Decode, and Why the Split Changes Everything

When I looked at my agent traces, task_4 cost $0.0963 and took 683 seconds while task_73 cost $0.0077 and took 43 seconds — a 14.7x cost difference. Same model. Same agent. Same reward outcome.

My baseline reported a clean average of $0.0199, as if the number were simple. It is not. Every LLM inference call splits into two fundamentally different phases — prefill and decode — and the ratio between them is what drives variance like this. Understanding the split changes how you read cost logs, design prompts, and predict p95 latency.

---

## The Load-Bearing Mechanism

Every time you call an LLM API, the work inside splits into two distinct phases:

**Phase 1 — Prefill**

All input tokens — system prompt, conversation history, user message — are processed in parallel in a single forward pass. The GPU runs one large matrix multiplication over the entire input at once. The output of this phase is not a response token; it is the KV cache: a stored representation of every input token's keys and values across all attention layers. Prefill is compute-bound — it uses the GPU's raw math throughput. Its latency is your Time to First Token (TTFT): how long before the model starts generating.

**Phase 2 — Decode**

Output tokens are generated one at a time, each in its own forward pass. To generate token N, the model attends over all previous tokens using the KV cache built in prefill. This is memory-bandwidth-bound — each step requires reading the full KV cache from GPU memory, not doing heavy matrix math. It cannot be parallelized across output tokens. Its latency per token is TPOT (Time Per Output Token), and total decode latency is TPOT × output token count.

```
Total inference time = TTFT (prefill) + TPOT × output_tokens (decode)
```

The phases face different hardware limits. Modern GPUs have far more compute (FLOPS) than memory bandwidth (GB/s). Prefill saturates compute — it is matrix multiplication, the GPU's strength. Decode saturates memory bandwidth — it reads the KV cache on every single token step, and the cache grows with context length. This is why a long decode is slow in a way that a long prefill is not: they hit different ceilings.

---

## Show It: Running the Analysis on Real Traces

I ran this analysis directly on 150 traces from my Week 10 Conversion Engine agent:

```python
import json
from collections import defaultdict

traces = []
with open("trace_log.jsonl") as f:
    for line in f:
        traces.append(json.loads(line.strip()))

# Group by task_id to see cost variance across 5 trials of the same task
by_task = defaultdict(list)
for t in traces:
    by_task[t["task_id"]].append((t["agent_cost"], t["duration"]))

# Show same-task cost variance
for task_id, runs in sorted(by_task.items()):
    costs = [r[0] for r in runs]
    ratio = max(costs) / min(costs)
    if ratio > 2.0:
        print(f"task_{task_id}: min=${min(costs):.4f} max=${max(costs):.4f} ratio={ratio:.1f}x")
```

**Output:**

```
task_4:   min=$0.0229  max=$0.0963  ratio=4.2x
task_11:  min=$0.0082  max=$0.0192  ratio=2.3x
task_83:  min=$0.0089  max=$0.0186  ratio=2.1x
task_105: min=$0.0140  max=$0.0998  ratio=7.1x
```

The same task, run 5 times, can cost 4–7x more on one trial than another. This is not noise — it is the agent taking different paths through its multi-turn tool-call loop, generating different amounts of output on each pass. Each additional tool call appends tokens to the input context (growing prefill cost on the next turn) and generates new output (adding decode latency).

**The cost-duration decoupling:**

```python
# High duration but low cost = agent was WAITING, not generating tokens
for t in sorted(traces, key=lambda x: x["duration"]/x["agent_cost"], reverse=True)[:5]:
    print(f"task_{t['task_id']}: cost=${t['agent_cost']:.4f}  duration={t['duration']:.1f}s")
```

**Output:**

```
task_92: cost=$0.0124  duration=687.3s
task_92: cost=$0.0125  duration=682.1s
task_95: cost=$0.0225  duration=723.1s
task_87: cost=$0.0295  duration=795.0s
task_87: cost=$0.0330  duration=827.2s
```

Task_92 spent 687 seconds at a cost of only $0.0124. No inference system generates that few tokens in 687 seconds — the agent was waiting for an external action, not running prefill or decode. **This proves that high p95 latency in an agent trace is not always driven by token generation.** Cost and latency can decouple completely when the agent is waiting.

**The quantitative shift using actual API pricing:**

```python
INPUT_PRICE  = 3.00 / 1_000_000   # $ per input token (prefill)
OUTPUT_PRICE = 15.00 / 1_000_000  # $ per output token (decode)

scenarios = [
    ("Short task (task_73 level)",  2_500,  150),
    ("Average task",                6_000,  400),
    ("Long task (task_4 level)",   10_000,  800),
    ("Double prompt only",         20_000,  800),
    ("Double output only",         10_000, 1_600),
]

for name, inp, out in scenarios:
    pc = inp * INPUT_PRICE
    dc = out * OUTPUT_PRICE
    print(f"{name}: prefill=${pc:.4f}  decode=${dc:.4f}  total=${pc+dc:.4f}")
```

**Output:**

```
Short task (task_73 level):  prefill=$0.0075  decode=$0.0023  total=$0.0098
Average task:                prefill=$0.0180  decode=$0.0060  total=$0.0240
Long task (task_4 level):    prefill=$0.0300  decode=$0.0120  total=$0.0420
Double prompt only:          prefill=$0.0600  decode=$0.0120  total=$0.0720
Double output only:          prefill=$0.0300  decode=$0.0240  total=$0.0540
```

The pattern is clear:

- **Double prompt length** → doubles prefill cost, raises Time to First Token only
- **Double output length** → doubles decode cost AND doubles total decode time — this is the p95 driver
- **Add a tool call turn** → prefill cost grows on the next turn, plus a new full decode cycle is added

**Output length drives p95 latency. Prompt length drives cost creep.**

In a multi-turn loop, each tool call compounds both: it appends to input context (raising next-turn prefill) and generates new output (adding decode time). This is the mechanism behind task_4's 683-second runtime.

---

## Connecting the Dots

**1. Why this is the prerequisite to understanding prefix caching**

The prefill phase builds the KV cache. If your agent re-sends the same system prompt on every API call (as most do), it is re-running prefill on that repeated prefix — unless the API's prefix caching applies. Understanding that prefill has a cost and that its output is the KV cache is what makes prefix caching intelligible. Without this, "prefix caching saves money" is just a marketing claim you cannot verify.

**2. Why speculative decoding targets decode, not prefill**

Prefill is already parallel — all input tokens in one pass. There is nothing to speed up by speculation. Decode is the bottleneck: sequential, memory-bandwidth-bound, one token per pass. Speculative decoding uses a small draft model to propose several candidate tokens in parallel, then verifies them with one full-model prefill-like pass. It works precisely because of the decode bottleneck this post describes.

**What I scoped out:** This post focuses on single-node inference. Multi-GPU tensor parallelism changes the memory bandwidth ceiling but not the fundamental prefill/decode distinction. Continuous batching changes how requests are scheduled across the two phases, but not the per-request breakdown.

---

## Pointers

**Papers read:**

1. Kwon et al., *Efficient Memory Management for Large Language Model Serving with PagedAttention*, SOSP 2023 (https://arxiv.org/abs/2309.06180) — the vLLM paper. Explicitly models and schedules prefill and decode as distinct phases.

2. Pope et al., *Efficiently Scaling Transformer Inference*, MLSys 2023 (https://arxiv.org/abs/2211.05102) — Google's analysis of why transformer inference is memory-bandwidth-bound during decode and compute-bound during prefill.

**Tool used:** Direct analysis of trace_log.jsonl from a real Week 10 agent — 150 traces, 30 tasks × 5 trials. All code above is runnable against the same file.

**Follow-on directions:**
- Continuous batching (Orca, OSDI 2022): how production inference servers interleave prefill and decode requests to maximize GPU utilization
- Chunked prefill: splitting long prefill passes into smaller chunks to reduce Time to First Token without sacrificing throughput

---

*Written as part of 10 Academy TRP1 Week 12 — closing inference-time mechanics gaps grounded in real agent traces.*
