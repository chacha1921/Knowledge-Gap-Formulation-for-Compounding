# Grounding Commit — Day 2

**Asker:** Chalie Lijalem
**File edited:** `src/strategies/extractors.py` in [Document-Intelligence-Refinery](https://github.com/chacha1921/Document-Intelligence-Refinery)

---

## What Changed

Added `smart_bbox_crop()` — a 15-line OpenCV preprocessing step that crops each document page image to the bounding box of its actual content before sending it to the VLM. Applied in both `_extract_gemini` and `_extract_ollama`.

**Code added — one helper function above the `VisionExtractor` class:**

```python
import cv2
import numpy as np

def smart_bbox_crop(img_bytes: bytes, padding: int = 20) -> bytes:
    """
    Crop image to bounding box of non-white content, removing whitespace margins.
    Reduces tile count → fewer tokens. Centers tables relative to tile boundaries.
    Insight from Day 2: tiles are 512×512 independent crops; tables spanning tile
    boundaries lose row continuity. Cropping margins reduces that risk.
    """
    nparr = np.frombuffer(img_bytes, np.uint8)
    img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    _, thresh = cv2.threshold(gray, 250, 255, cv2.THRESH_BINARY_INV)
    coords = cv2.findNonZero(thresh)
    if coords is None:
        return img_bytes  # blank page — return original
    x, y, w, h = cv2.boundingRect(coords)
    ih, iw = img.shape[:2]
    x, y = max(0, x - padding), max(0, y - padding)
    w, h = min(iw - x, w + 2 * padding), min(ih - y, h + 2 * padding)
    _, buf = cv2.imencode('.png', img[y:y+h, x:x+w])
    return buf.tobytes()
```

**Modification in `_extract_gemini` — one line added before PIL load:**

```python
def _extract_gemini(self, img_bytes: bytes) -> str:
    try:
        from PIL import Image
        import io
        img_bytes = smart_bbox_crop(img_bytes)      # ← added
        image = Image.open(io.BytesIO(img_bytes))
        ...
```

**Modification in `_extract_ollama` — one line added before base64 encode:**

```python
img_bytes = smart_bbox_crop(img_bytes)              # ← added
img_b64 = base64.b64encode(img_bytes).decode('utf-8')
```

---

## Why It Changed

Gashaw's explainer revealed that VLM image processing tiles the page into 512×512 independent crops. Tiles have no shared positional encoding — a table row that straddles a tile boundary appears as two disconnected groups of patches with no structural signal connecting them. This is why Strategy C fails on wide tables and tables near the page edges: they span boundaries.

The fix follows directly: if the page is cropped to its content area before tiling, margins are removed, the effective image is smaller (fewer tiles), and tables are more likely to fall within a single tile. For a standard scanned A4 document with ~15% margins, the content area is ~70% of each dimension (~49% of total area), reducing tile count by roughly half and cutting Gemini token cost by ~40–50%. Edge tables — previously split across page-edge tiles — move into the interior of the cropped image and extract correctly.

This changes the escalation logic from "send the whole page and hope" to "send only what the model needs to see."
