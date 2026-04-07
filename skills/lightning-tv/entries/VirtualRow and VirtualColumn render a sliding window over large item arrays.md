---
description: Both components accept a full `each` array but only mount `displaySize + bufferSize` items, shifting the window as the user navigates
type: architecture
module: primitives
created: 2026-04-07
---

`VirtualRow` and `VirtualColumn` are the primary virtualization primitives for 1D lists in Lightning TV. They share a single `createVirtual` factory, parameterized by whether they use Row or Column as the underlying navigable container. The `each` prop holds the full dataset; only a slice of it is ever rendered.

## Required Props

```ts
displaySize: number  // how many items fit on screen simultaneously
each: readonly T[]   // the full dataset (can be large)
```

`bufferSize` (default: 2) adds extra items beyond `displaySize` to the rendered slice so the next items are already mounted when the user navigates to them.

## How the Window Works

At any moment, the rendered slice contains `displaySize + bufferSize` items. As navigation moves the cursor through `each`, the slice start index shifts to keep the cursor visible. The component tracks two positions simultaneously:

- `cursor`: absolute index in the full `each` array
- `selected`: position within the current rendered slice

Since [[VirtualRow and VirtualColumn track absolute cursor and relative selected as separate signals]], these two values can diverge and must be kept in sync on every navigation event.

## Scroll Modes

The `scroll` prop (inherited from RowProps) selects the windowing strategy:

| Mode | Behavior |
|------|----------|
| `'auto'` | Selection moves within the window first; window scrolls once selection pins to `scrollIndex` |
| `'always'` | Window scrolls immediately; selection stays pinned at `bufferSize` |
| `'edge'` | Selection moves until it reaches the edge of the visible area; then window scrolls |
| `'none'` | Window stays at 0; selection moves freely (no virtualization) |

## Wrap Mode

Setting `wrap: true` enables circular navigation. The slice is built using modular indexing so the first item follows the last. The container is initially offset by `-1 * childSize` so one item is always visible before the focused item, making wrap transitions seamless.

## Key Handlers

`VirtualRow` chains user `onLeft`/`onRight` with `lngp.handleNavigation`. `VirtualColumn` chains `onUp`/`onDown`. User-provided handlers fire first; framework navigation runs if they don't return true.

---

Related Entries:
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — VirtualRow wraps Row as its underlying navigable container; all Row props including scroll mode and transitions are inherited
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — VirtualColumn wraps Column; the y-axis flex layout is what makes the windowing translate to visual position changes
- [[Virtual component uses adaptive animation duration to handle rapid navigation]] — the adaptive animation mechanism that prevents queuing buildup during fast navigation in virtual lists
- [[createInfiniteItems accumulates paginated data into a single reactive array]] — the canonical data primitive pairing; Virtual provides the window, createInfiniteItems provides the growing array
- [[Virtual wrap mode uses modular indexing to build a circular slice from any start position]] — extends this by documenting the circular navigation implementation when wrap:true is set

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
