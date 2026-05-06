# Sources — Day 2

**Explainer written by:** Chalie Lijalem
**Question answered:** Gashaw Bekele's question on tool selection mechanism in function-calling agents

---

## Primary Sources Read

### 1. ReAct — Synergizing Reasoning and Acting (load-bearing)
**Citation:** Yao et al., *ReAct: Synergizing Reasoning and Acting in Language Models*, ICLR 2023 (https://arxiv.org/abs/2210.03629)
**Why canonical:** The paper that made reasoning-over-descriptions visible. By forcing models to output explicit reasoning traces before each action, it showed that models do reason about tool descriptions before selecting — the model generates thoughts like "I need X, tool Y handles X" before committing to a call. This is the primary evidence that tool selection is reasoning-driven, not random.
**How it is load-bearing:** The claim "the model reasons over tool descriptions before selecting" and the characterization of selection as reasoning-driven (not a separate routing module) is supported by ReAct's analysis of reasoning traces.

### 2. Toolformer — Language Models Can Teach Themselves to Use Tools (load-bearing)
**Citation:** Schick et al., *Toolformer: Language Models Can Teach Themselves to Use Tools*, NeurIPS 2023 (https://arxiv.org/abs/2302.04761)
**Why canonical:** Shows that models learn *when and which* tool to invoke from few demonstrations embedded in the language modeling objective — not from explicit "call this tool when X" supervision. This explains where the fine-tuning behavior that drives tool selection comes from.
**How it is load-bearing:** The mechanism claim that "the model was fine-tuned on function-calling examples and learned to read tool schemas" traces back to Toolformer's analysis of how tool selection emerges from the training objective.

### 3. ToolScope — Tool Merging and Context-Aware Filtering (load-bearing)
**Citation:** Liu et al., *ToolScope: Enhancing LLM Agent Tool Use through Tool Merging and Context-Aware Filtering*, arXiv 2025 (https://arxiv.org/abs/2510.20036)
**Why canonical:** The most directly relevant paper to Gashaw's question. Specifically measures the impact of overlapping tool descriptions on selection accuracy across multiple benchmarks. Introduces the concept of "semantically redundant tools with overlapping names and descriptions" as a documented failure mode.
**How it is load-bearing:** The 8.8–38.6% degradation figure and the claim that overlapping descriptions are a correctness bug (not a performance issue) come from this paper's benchmark results.

### 4. Disambiguation-Centric Finetuning (supporting)
**Citation:** Hathidara et al., *Disambiguation-Centric Finetuning Makes Enterprise Tool-Calling LLMs More Realistic and Less Risky*, ACL 2026 (https://arxiv.org/abs/2507.03336)
**Why relevant:** Studies the hard case — near-duplicate tools and underspecified user intent — and shows that the right response is to generate clarifying questions rather than guess. The 27 pp improvement over GPT-4o establishes that disambiguation is a learnable behavior, not just a prompt engineering concern.
**How it is load-bearing:** The "connecting the dots" section's claim about when descriptions cannot disambiguate (and what to do then) is grounded in this paper's findings.

---

## API Documentation Used

- **Anthropic Tool Use Documentation** — https://docs.anthropic.com/en/docs/build-with-claude/tool-use  
  Used for: the exact serialization format (what tools look like to the model), schema validation behavior (Layer 2 constraint), and the `cache_control` parameter for caching tool definitions.

- **OpenAI Function Calling Documentation** — https://platform.openai.com/docs/guides/function-calling  
  Used for: the original JSON Schema format for tool definitions; the `tool_choice` parameter for controlling selection behavior.

---

## What I Did Not Use

- **Gorilla** (Patil et al., NeurIPS 2024) — relevant to the adjacent question of tool RETRIEVAL (selecting from hundreds of APIs), but not directly relevant to the mechanism of selection among a small set of tools in context. Not cited as load-bearing.
- **ToolLLM / ToolBench** (Qin et al., ICLR 2024) — studies tool selection at scale (16,000+ APIs) with a neural retriever. Relevant to the large-scale case but not to the mechanism question. Not load-bearing.
- Second-hand blog posts about "prompt engineering for tool use" — these propagate the misconception that tool selection is a framework constraint rather than a model behavior. Not used.
