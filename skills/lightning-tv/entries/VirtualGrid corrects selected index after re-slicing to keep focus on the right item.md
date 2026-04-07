---
description: After a row change causes the slice window to shift, VirtualGrid applies an `idxCorrection` to `this.selected` so focus lands on the correct child in the new slice
type: gotcha
module: primitives
created: 2026-04-07
---

When VirtualGrid's window shifts (triggered by vertical navigation to a new row), the rendered slice changes. The underlying navigable element's `this.selected` is a position within the OLD slice. After `setSlice` updates the children, that position now points to the wrong item.

## The Correction

```ts
const idxCorrection = prevStart - start();
if (lastIdx) lastIdx += idxCorrection;
idx += idxCorrection;
this.selected += idxCorrection;
```

`prevStart - start()` is the number of item positions the window moved. Subtracting it from `this.selected` keeps the same physical item focused even though its array position within the slice changed.

This correction is applied BEFORE calling `columnScroll` (the vertical scroll handler), so scroll animates to the right child.

## Why This Is Easy to Get Wrong

The framework's built-in selection tracking (Row/Column's `selected` property) does not know the slice changed. It still holds the pre-shift index. Any code that reads `this.selected` after a slice update — without applying the correction — will operate on a stale value. This includes any user-provided `onSelectedChanged` handlers chained before the internal one.

## Selected Prop on Initial Render

When `props.selected` is non-zero on first render, `onCreate` chains `columnScroll` to scroll to the initial position. Without this, the grid would render visually at the top even though the selected item might be mid-list.

---

Source: [[primitives-VirtualGrid]]
Domains:
- [[performance guide]]
- [[components guide]]
