---
description: Grid.scrollToIndex clamps the index, focuses the grid container if it doesn't have focus, updates focusedIndex reactively, then queues child focus as a microtask
type: api
module: primitives
created: 2026-04-07
---

# Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus

Grid exposes `scrollToIndex` on its element node with distinct behavior compared to [[Row and Column scrollToIndex imperatively sets selected scrolls and focuses without respecting scroll=none]]:

```ts
function scrollToIndex(this: ElementNode, index: number) {
  untrack(() => {
    if (!props.items || props.items.length === 0) return;

    if (!hasFocus(gridRef)) {
      gridRef.setFocus();  // Focus the GRID first
    }

    const clampedIndex = Math.max(0, Math.min(index, props.items.length - 1));
    setFocusedIndex(clampedIndex);    // Update signal reactively
    queueMicrotask(focus);            // Focus child on next tick
  });
}
```

Key differences from Row/Column's `scrollToIndex`:

1. **Grid self-focuses first**: If the grid doesn't already have focus, it calls `gridRef.setFocus()` before anything else. Row/Column never self-focus in `scrollToIndex`.

2. **Reactive index update**: Uses `setFocusedIndex(clampedIndex)` — a Solid signal setter — which triggers the `scrollY` memo to recompute and scroll the grid.

3. **Microtask deferral**: `queueMicrotask(focus)` defers the child `setFocus` call to allow reactive updates to settle. The child may not exist at the new position until the next render tick.

4. **Index clamping**: `Math.max(0, Math.min(index, items.length - 1))` — invalid indices are clamped silently. Row/Column do no clamping.

5. **Wrapped in `untrack`**: Prevents this function from being called inside a reactive context that might re-trigger it.

**Side effect alert:** Calling `scrollToIndex` on a Grid that doesn't have focus will STEAL focus to the grid. This is a significant side effect if called from a currently-focused component. The `gridRef.setFocus()` call routes through [[setFocus defers focus assignment via microtask to allow children to render first]], while the child focus also defers via `queueMicrotask` — meaning two separate microtask ticks are required before the target child has focus.

---

Related Entries:
- [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]] — explains why Grid uses a signal for focusedIndex and why the reactive update is needed before focusing the child

Source: [[primitives-Grid]]
Domains:
- [[navigation guide]]
