# My Question — Day 2

**Subtopic:** Vision-Language Model image encoding mechanics; what `detail: high` actually does; how spatial document structure (tables, forms) survives patch encoding.

**Artifact referenced:** `week3/Document_Intelligence_Refinery/` — Strategy C (Vision-Augmented) fallback layer; the routing logic that escalates failed pages to GPT-4o-mini with `detail: "high"`

---

## Draft Question (before morning call)

> In my Week 3 Document Intelligence Refinery, I implemented the required 'Strategy C (Vision-Augmented)' fallback layer. When my FastText or Layout heuristics fail on complex tables or scanned forms, my pipeline escalates the entire document image directly to GPT-4o-mini using the `detail: high` parameter. Right now, my routing logic treats the Vision-Language Model as a magic black box that just 'looks' at the whole page, but I don't actually understand the mathematical or architectural mechanics of how it processes that image.

---

## Final Question (after morning call sharpening)

> In my Week 3 Document Intelligence Refinery, my Strategy C fallback escalates failed document pages to GPT-4o-mini with `detail: high`. The VLM correctly extracts structure from some complex tables but fails on others in ways I cannot predict. The specific gap: I don't know what `detail: high` actually does to the image before it reaches the language model — how many tiles does it create, what is the token cost per page, what architectural mechanism encodes the image patches into the language model's token space, and what information from a table's spatial layout (column headers, cell boundaries, row structure) survives or is lost in that encoding? Without understanding this, I cannot predict when Strategy C will succeed or fail, cannot estimate cost per page, and cannot design a smarter escalation trigger than "try it and see."

---

## Connection to Existing Work

- **File:** `week3/Document_Intelligence_Refinery/` — Strategy C routing logic, the `detail: high` API call
- **Language I used without understanding:** "escalates to vision model for complex layout" — I wrote this implying I knew what "vision processing" means mechanically. I do not.
- **Specific unverified assumption:** I assumed `detail: high` means "higher quality" without knowing it changes the tiling resolution, the token count, and therefore the cost per escalated page.
- **What would change if gap is closed:**
  1. I can calculate the token cost of every escalated page (and whether `detail: low` is sufficient for most cases)
  2. I can predict which documents will fail Strategy C (e.g., tables wider than a single tile, or forms with fine-grained spatial alignment)
  3. I can design the escalation trigger based on document properties that predict VLM success, not just "did the upstream layers fail?"

---

## Gap Triage

- [x] Non-trivial (not a lookup)
- [x] Connected to a specific existing artifact
- [x] Closure would change something concrete in my work
- [x] Has a satisfying answer
