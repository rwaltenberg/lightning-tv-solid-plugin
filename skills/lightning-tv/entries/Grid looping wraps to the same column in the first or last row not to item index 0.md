---
description: When Grid looping=true, navigating past the end wraps to the same column in the first row, and past the start wraps to the same column in the last row
type: gotcha
module: primitives
created: 2026-04-07
---

# Grid looping wraps to the same column in the first or last row not to item index 0

Grid's `looping` prop wraps navigation at the boundaries, but unlike Row/Column `wrap`, it wraps to the same COLUMN rather than to index 0 or the last index.

**Looping forward (down past last row):**
```ts
setFocusedIndex(focusedIndex() % columns());
```
If focused index was column 2 in row 3, wraps to index 2 (column 2 in row 0).

**Looping backward (up past first row):**
```ts
const lastRowStart = totalItems - (totalItems % columns()) || totalItems - columns();
const target = lastRowStart + (focusedIndex() % columns());
setFocusedIndex(target < totalItems ? target : target - columns());
```
Wraps to the same column in the last row. The `|| totalItems - columns()` handles the case where `totalItems % columns() === 0` (a perfectly filled grid — `totalItems - 0` would be out of bounds).

**Horizontal looping within a row:**
```ts
const rowStart = Math.floor(focusedIndex() / columns()) * columns();
const rowEnd = Math.min(rowStart + columns() - 1, props.items.length - 1);
setFocusedIndex(delta > 0 ? (newIndex > rowEnd ? rowStart : newIndex) : newIndex < rowStart ? rowEnd : newIndex);
```
Wraps within the current row only — going right past the last column returns to rowStart, going left past the first column returns to rowEnd.

**Contrast with Row/Column `wrap`:** Row/Column `wrap` wraps from last item to index 0 and vice versa, crossing row boundaries. Grid `looping` preserves column alignment.

Since [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]], the column count is fixed by `columns` prop, making column-preserving wrap straightforward.

---

Source: [[primitives-Grid]]
Domains:
- [[navigation guide]]
