# Call Summary — Day 2

**Date:** 2026-05-07
**Topic:** Tool selection in function-calling agents
**Pair:** Chalie Lijalem + Gashaw Bekele

---

## Questions Before Sharpening

**Gashaw's draft question:**
> How Tool Selection Actually Works in Function-Calling Agents — when an LLM agent has multiple tools available, what is the mechanism by which it selects one — does the model reason over tool descriptions, does the framework constrain the output format, or both? Specifically, when two tools serve overlapping purposes (like a semantic search tool and a structured query tool), what determines which one the model picks, and what makes a tool description reliably steer that choice?

**Chalie's draft question:**
> In my Week 3 Document Intelligence Refinery, I implemented the required 'Strategy C (Vision-Augmented)' fallback layer. When my FastText or Layout heuristics fail on complex tables or scanned forms, my pipeline escalates the entire document image directly to GPT-4o-mini using the `detail: high` parameter. Right now, my routing logic treats the Vision-Language Model as a magic black box that just 'looks' at the whole page, but I don't actually understand the mathematical or architectural mechanics of how it processes that image.

---

## What Was Interrogated and How Questions Moved

1. **Chalie pushed back on Gashaw's draft:** The question has two parts that could be separated — the mechanism of selection (how the model decides) vs. what makes descriptions reliable (an engineering prescription). Gashaw confirmed both parts are load-bearing: you can't answer the second without grounding it in the first. The question stayed as a compound but with the mechanism part leading.

2. **Gashaw pushed back on Chalie's draft:** "What specific failure are you pointing at — are you asking why Strategy C fails on some tables but not others, or are you asking what the mechanism is so you can predict failure in advance?" Chalie identified the real gap: the routing logic has no predictive model of when the VLM will fail because it doesn't understand what the VLM can and cannot see. The question sharpened from "how does it work mechanically" to "what spatial information from a document's layout survives the encoding, and what does `detail: high` actually do to the image?"

3. **Final direction confirmed:** Gashaw's question anchors on the mechanism (model reasoning vs. framework constraints) and the overlap case specifically — concrete enough to produce a grounded explainer with a before/after description example. Chalie's question anchors on VLM image encoding mechanics — the tiling, token cost, and information preservation question — specific enough to close with a calculation and a concrete prediction about Strategy C failure cases.

---

## Final Questions (committed after call)

**Gashaw's final question:**
> When an LLM agent has multiple tools available, what is the mechanism by which it selects one — does the model reason over tool descriptions, does the framework constrain the output format, or both? Specifically, when two tools serve overlapping purposes (like a semantic search tool and a structured query tool), what determines which one the model picks, and what makes a tool description reliably steer that choice?

**Chalie's final question:**
> In my Week 3 Document Intelligence Refinery, my Strategy C fallback escalates failed document pages to GPT-4o-mini with `detail: high`. The VLM correctly extracts structure from some complex tables but fails on others in ways I cannot predict. The specific gap: I don't know what `detail: high` actually does to the image before it reaches the language model — how many tiles does it create, what is the token cost per page, what architectural mechanism encodes the image patches into the language model's token space, and what information from a table's spatial layout (column headers, cell boundaries, row structure) survives or is lost in that encoding? Without understanding this, I cannot predict when Strategy C will succeed or fail, cannot estimate cost per page, and cannot design a smarter escalation trigger than "try it and see."

---

## Attestation
- [x] Gashaw attests Chalie's question is unambiguous
- [x] Chalie attests Gashaw's question is unambiguous
