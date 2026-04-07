---
description: The edge scroll mode checks the renderer's renderState property (InViewPort===8) to determine if the next item is visible before deciding to scroll the container
type: architecture
module: primitives
created: 2026-04-07
---

# edge scroll mode uses renderer viewport state to detect off-screen items before scrolling

The `edge` scroll mode in [[withScrolling implements six scroll modes for navigable list containers]] works by checking whether the NEXT item (one past the currently selected) is visible in the renderer's viewport before deciding to scroll.

```ts
const InViewPort = 8;
const isNotShown = (node: ElementNode | ElementText) => {
  return node.lng.renderState !== InViewPort;
};

// In edge mode:
else if (isIncrementing && isNotShown(nextElement)) {
  nextPosition = rootPosition - selectedSize - gap;
} else if (isNotShown(nextElement)) {
  nextPosition = -selectedPosition + offset;
}
```

When moving forward: if the next item is off-screen, shift the container by one item width/height.
When moving backward: if the previous item (nextElement when decrementing) is off-screen, snap the selected item to the starting edge.

The `InViewPort = 8` constant is hardcoded from the Lightning renderer's `renderState` enum. It is not exported from the renderer — this is an implementation detail that could break if the renderer changes its state numbering.

**Behavioral contrast with `auto`:**
- `auto` scrolls immediately on every navigation press (once past `scrollIndex`)
- `edge` holds position until the boundary is reached, then scrolls

**Use case:** `edge` keeps items visible longer in a fixed viewport, only shifting when absolutely necessary. Useful for short lists where you want to show as many items as possible without unnecessary movement.

---

Source: [[primitives-withScrolling]]
Domains:
- [[navigation guide]]
