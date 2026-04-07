---
description: Text nodes with contain 'width' or 'none' use getBoundingClientRect() divided by DPR and parent scale to update their w/h props, then emit 'loaded' to trigger layout recalculation
type: architecture
module: core
created: 2026-04-07
---

# DOMText measures its own dimensions after render to feed back into flex layout

In the DOM renderer, text dimensions are not known until the browser has laid out the text. `DOMText` handles this with a deferred measurement system.

**Measurement is needed for:**
- `contain: 'width'` — browser wraps text; actual height is unknown until rendered
- `contain: 'none'` — text expands freely; both width and height are unknown

After every change to font properties or text content, `scheduleUpdateDOMTextMeasurement(node)` is called. This defers measurement via `setTimeout` (after fonts have loaded via `document.fonts.ready`).

Measurement uses `getBoundingClientRect()` and corrects for scale transforms up the parent chain:

```ts
function getElSize(node: DOMNode): Size {
  const rawRect = node.div.getBoundingClientRect();
  const dpr = Config.rendererOptions?.deviceLogicalPixelRatio ?? 1;
  let width = rawRect.width / dpr;
  let height = rawRect.height / dpr;
  // Walk up parent chain and divide out scale
  for (;;) {
    if (node.props.scale != null && node.props.scale !== 1) {
      width /= node.props.scale; height /= node.props.scale;
    } else {
      width /= node.props.scaleX; height /= node.props.scaleY;
    }
    if (node.parent instanceof DOMNode) node = node.parent;
    else break;
  }
  return { width, height };
}
```

After measurement, if dimensions changed, the node emits `'loaded'` with the new dimensions. This event triggers flex recalculation in the parent container.

Font loading integration: `setupFontLoadingListeners()` registers a `loadingdone` listener on `document.fonts`. When fonts finish loading, all tracked `containTextNodes` are re-measured. This handles the common case where font load changes text dimensions after initial layout.

Since [[DOMNode renders Lightning visual properties as computed inline CSS on a div element]], `DOMText` inherits the full style rendering pipeline, and text-specific properties (fontSize, fontFamily, letterSpacing, etc.) are all applied via `updateNodeStyles()`.

---

Source: [[dom-renderer]]
Domains:
- [[core guide]]
