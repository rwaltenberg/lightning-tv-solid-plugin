---
description: Grid.onSelectedChanged is (index, grid, elm?) while NavigableProps.onSelectedChanged is (index, elm, child, lastIdx) — the argument order and count differ, causing silent bugs if shared
type: gotcha
module: primitives
created: 2026-04-07
---

# Grid onSelectedChanged has a different signature than NavigableProps onSelectedChanged

Grid defines its own `onSelectedChanged` signature in `GridProps`:

```ts
// Grid's onSelectedChanged:
onSelectedChanged?: (index: number, grid: ElementNode, elm?: ElementNode) => void;
```

Row and Column use `NavigableProps.OnSelectedChanged`:
```ts
// Row/Column's onSelectedChanged (from OnSelectedChanged type):
onSelectedChanged?: (
  this: NavigableElement,
  selectedIndex: number,
  elm: NavigableElement,
  active: ElementNode,
  lastSelectedIndex?: number,
) => void;
```

The differences:
1. Grid: `(index, grid, elm?)` — `elm` is optional, no `lastSelectedIndex`
2. Row/Column: `(index, elm, child, lastIdx?)` — second arg is the container, third is the focused child, fourth is previous index

In Grid's implementation, `onSelectedChanged` is called as:
```ts
props.onSelectedChanged?.call(gridRef, focusedIndex(), gridRef, focusedElm);
```

The `this` context is `gridRef` but the function signature doesn't declare `this`, unlike `NavigableProps.OnSelectedChanged` which declares `this: NavigableElement`.

**Consequence:** A handler written for Row/Column will receive different arguments when placed on a Grid. The `lastSelectedIndex` parameter is always `undefined` on Grid. If you reuse a handler across Row/Column and Grid, the third argument's semantics differ — for Row/Column it's the focused child element, for Grid it's the focused child element too, but Grid's `elm` is optional.

---

Source: [[primitives-Grid]]
Domains:
- [[navigation guide]]
- [[components guide]]
