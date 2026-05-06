# Tweet Thread — Day 2

**Topic:** How Tool Selection Works in Function-Calling Agents
**Question answered for:** Gashaw Bekele

---

**Tweet 1/5**

Tool selection in a multi-tool LLM agent isn't routing logic or a special module.

It's the model reading text and doing next-token prediction.

Here's the mechanism — and why overlapping descriptions silently break things with no error 🧵

---

**Tweet 2/5**

Both happen — but at different layers:

→ Layer 1 (model): reasons over descriptions to pick a tool
→ Layer 2 (framework): enforces valid JSON format after selection

Layer 2 can't catch wrong selection. You can call the wrong tool with perfectly valid JSON.

---

**Tweet 3/5**

When two tools overlap, what steers selection:

1. Anti-triggers ("NOT for exact IDs") — most powerful
2. Trigger phrases ("use for similarity")
3. Tool name clarity
4. Parameter overlap with the user message

ToolScope (2025): 8–38% accuracy loss from overlapping descriptions.

---

**Tweet 4/5**

The reliable description pattern:

1. Trigger: "Use when you have an exact field value"
2. Examples: "ID, email, SKU, date range"
3. Anti-trigger: "NOT for fuzzy queries"

Anti-triggers beat trigger phrases. They create active suppression, not just lower weight.

---

**Tweet 5/5**

Test your tool descriptions before shipping:

Run each overlapping pair against 6–10 messages with expected calls.
100% = unambiguous. Any miss = description problem, not model problem.

Fix descriptions before fine-tuning or retrieval. This is a correctness bug.

🔗 https://medium.com/@chalielijalem/how-tool-selection-actually-works-in-function-calling-agents-1961c1d3d222
