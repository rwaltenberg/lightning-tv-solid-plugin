---
description: VirtualGrid renders a windowed subset of a 2D dataset using flexWrap, aligning slice boundaries to column counts so rows are never split
type: architecture
module: primitives
created: 2026-04-07
---

`VirtualGrid` is the 2D equivalent of VirtualRow/VirtualColumn. It renders a flat array as a grid by combining `flexWrap: 'wrap'` layout with row-aware windowing. Items flow naturally into rows based on `columns` (items per row), and the rendered window is always aligned to row boundaries.

## Props

```ts
columns: number   // Required: items per row
rows?: number     // Visible rows (default: 1)
buffer?: number   // Buffer rows above and below (default: 2)
```

## Slice Calculation

```ts
const start = createMemo(() => {
  const rowIndex = Math.floor(cursor() / columns);
  return Math.max(0, (rowIndex - buffer) * columns);
});
const end = createMemo(() => {
  const rowIndex = Math.floor(cursor() / columns);
  return Math.min(items.length, (rowIndex + buffer) * columns + totalVisibleItems);
});
```

`totalVisibleItems = columns * rows`. The window always starts and ends on a column boundary, so the flexWrap grid is never visually broken (partial rows only occur at the actual end of the dataset).

## Navigation

Horizontal navigation (left/right) uses `navigableHandleNavigation` — standard sibling-based navigation within the flat item list. Vertical navigation is custom:

```ts
function onVerticalNav(dir: -1 | 1) {
  const offset = dir * columns;  // jump by one full row
  const newIndex = clamp(selected + offset, 0, items.length - 1);
}
```

After vertical navigation, the component calls `chainedOnSelectedChanged` manually to trigger slice updates and scrolling.

## Position Preservation During Re-slice

When the selected row changes and the slice is updated, the component preserves the visual Y position via a microtask:

```ts
const prevRowY = this.y + active.y;
this.updateLayout();
this.lng.y = prevRowY - active.y;
columnScroll(idx, elm, active, lastIdx);
```

This prevents a visible jump when the flex layout recalculates item positions after the slice changes.

## Scroll Mode

VirtualGrid always uses `scroll: 'always'` regardless of what the prop says. Horizontal scrolling within a row uses `navigableHandleNavigation`; vertical scrolling uses `columnScroll` (withScrolling(false)).

Since [[VirtualRow and VirtualColumn render a sliding window over large item arrays]], VirtualGrid uses the same onEndReached pattern for pagination.

Since [[flexWrap wraps children to new rows when they overflow the main axis]], VirtualGrid relies on the flex engine's wrap behavior to convert a flat array into a grid — `columns` and item widths determine when wrapping occurs.

---

Related Entries:
- [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]] — the non-virtual 2D alternative; Grid uses absolute positioning instead of flex layout, which is simpler but renders all items at once
- [[VirtualGrid corrects selected index after re-slicing to keep focus on the right item]] — extends this by documenting the internal index correction applied after row changes

Source: [[primitives-VirtualGrid]]
Domains:
- [[performance guide]]
- [[components guide]]
