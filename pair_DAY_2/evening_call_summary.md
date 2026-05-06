# Evening Call Summary — Day 2

**Date:** 2026-05-07
**Pair:** Chalie Lijalem + Gashaw Bekele
**Written by:** Chalie Lijalem

---

The explainer closed the gap. The two mechanisms — ViT patch embedding and OpenAI's tiling algorithm — were explained clearly and in the right order. The Python token calculator (85 + 170×tiles) gave an exact number I can use: 1,105 tokens and $0.000166 per standard scanned A4 page, which means latency, not cost, is the real constraint on Strategy C.

The one piece of feedback I gave Gashaw: the "why wide tables fail" section explains that tile boundaries cut through rows, but does not make explicit why positional encoding does not bridge the gap — tiles are independent crops and a patch at position (0,0) of tile 2 has no encoded positional relationship to any patch in tile 1. Gashaw added a clarifying sentence to that section.

The two-pass strategy (detail: low for classification, detail: high only for confirmed tables) was not something I had thought of — it came directly from understanding the token formula and is a concrete implication I will commit back to Week 3.
