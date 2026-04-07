---
description: Images store their src as lazyImagePendingSrc and only set the img element's src attribute when renderState becomes 4 (inBounds) or 8 (inViewport)
type: architecture
module: core
created: 2026-04-07
---

# DOM renderer defers image loading until node enters bounds using renderState tracking

The DOM renderer implements its own form of lazy image loading that mirrors the WebGL renderer's out-of-bounds texture culling.

`DOMNode` maintains:
- `lazyImagePendingSrc` — the URL waiting to be loaded
- `renderState` — one of: 0 (init), 2 (outOfBounds), 4 (inBounds), 8 (inViewport)
- `boundsDirty` — flag indicating position has changed and render state needs recalculation

When a node has an image source, `updateNodeStyles()` stores the URL but does NOT set `img.src` immediately:

```ts
node.lazyImagePendingSrc = rawImgSrc;
if (isRenderStateInBounds(node.renderState)) {
  node.applyPendingImageSrc();
}
```

`isRenderStateInBounds` returns true for states 4 and 8 only. `applyPendingImageSrc()` sets `img.src = pendingSrc` and tracks it in `img.dataset.rawSrc`.

**Bounds computation** in `computeRenderStateForNode()`:
- Calculates the root viewport rectangle
- Expands it by `boundsMargin` (a node-level or stage-level setting)
- Returns state 2 if the node doesn't intersect the expanded bounds
- Returns state 8 if it intersects the exact viewport
- Returns state 4 if it's in expanded bounds but not viewport

**Propagation**: when a node's position changes (x, y, w, h, scale, mount, parent), it sets `boundsDirty = true` and calls `markChildrenBoundsDirty()` recursively. The next call to `updateNodeStyles()` recomputes the render state via `computeRenderStateForNode()`.

Since [[DOMNode renders Lightning visual properties as computed inline CSS on a div element]] handles all style application, the render state check is integrated into the same `updateNodeStyles()` call path.

---

Source: [[dom-renderer]]
Domains:
- [[core guide]]
- [[performance guide]]
