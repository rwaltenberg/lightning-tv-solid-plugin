---
description: getElementScreenRect(el, from?, out?) walks the parent chain accumulating position and scale offsets, then applies deviceLogicalPixelRatio from Config — returns a Rect in screen pixels
type: api
module: core
created: 2026-04-07
---

# getElementScreenRect calculates absolute screen coordinates accounting for scale and DPR

`getElementScreenRect` computes the actual on-screen position and dimensions of an element by walking up the parent chain.

```ts
import { getElementScreenRect } from '@lightning/solid';

const rect = getElementScreenRect(myNode);
// { x: 120, y: 240, width: 300, height: 200 } — in screen pixels

// Relative to a specific ancestor:
const relativeRect = getElementScreenRect(myNode, containerNode);

// Avoid allocation by passing output object:
const out = { x: 0, y: 0, width: 0, height: 0 };
getElementScreenRect(myNode, undefined, out);
```

**Calculation details:**
1. Starts with element's own `width` and `height`, applying its own `scaleX`/`scaleY`
2. Walks up parent chain (stopping at `from` if provided, or root if not)
3. At each level, adds `x` and `y` and applies scale center offset: `(width / 2) * (1 - scaleX)`
4. After traversal, multiplies final values by `Config.rendererOptions.deviceLogicalPixelRatio`

**Scale origin is center:** When a parent is scaled, its children's effective positions shift because scale is applied from the center point. `getElementScreenRect` accounts for this with the offset formula.

**DPR application:** If `deviceLogicalPixelRatio` is set in `Config.rendererOptions`, all four values (x, y, width, height) are scaled by it. This ensures returned coordinates match physical screen pixels.

**`out` parameter:** Pass a pre-allocated `Rect` object to avoid GC pressure in frequent calls (e.g., during animations or focus moves).

Since [[Config singleton holds all runtime Lightning configuration]], `deviceLogicalPixelRatio` is read from `Config.rendererOptions` at call time — changing it after initialization will affect future `getElementScreenRect` calls.

---

Source: [[core-utils]]
Domains:
- [[core guide]]
