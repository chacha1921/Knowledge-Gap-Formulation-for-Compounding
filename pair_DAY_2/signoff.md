# Sign-off — Day 2

**Asker:** Chalie Lijalem
**Explainer written by:** Gashaw Bekele
**Gap-closure judgment:** Closed

---

Before this explainer, I treated the VLM in Strategy C as a black box that "looked at the whole page." I had no number for token cost, no understanding of what `detail: high` actually did, and no way to predict when the model would fail on a table.

I now understand: the image is tiled into 512×512 windows after a two-step resize, each tile costs 170 tokens plus an 85-token base overhead, and a standard scanned A4 page at 150 DPI produces 6 tiles (1,105 tokens, $0.000166). The budget is not the constraint — latency is. I also understand why wide tables fail: tile boundaries cut through rows, and because tiles are independent crops with no shared positional encoding between them, the model cannot reconstruct row continuity across a boundary. This was the prediction capability I was missing — I can now look at a table's width relative to tile size (~3.4 inches per tile) and predict whether Strategy C will succeed before making the API call. The two-pass strategy (detail: low for page classification, detail: high only for confirmed table/form pages) is a direct actionable implication I will commit back to my Week 3 pipeline.
