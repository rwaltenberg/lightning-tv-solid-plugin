---
description: withScrolling(isRow) produces a Scroller that repositions a Row or Column container based on one of six scroll modes — auto, edge, always, center, bounded, or none
type: architecture
module: primitives
created: 2026-04-07
---

# withScrolling implements six scroll modes for navigable list containers

`withScrolling(isRow: boolean)` is a factory that creates a scroll function for navigable containers. `scrollRow = withScrolling(true)` operates on the x-axis; `scrollColumn = withScrolling(false)` operates on the y-axis. Both are attached to Row/Column's `onSelectedChanged` handler.

The public `NavigableProps.scroll` type exposes five modes: `'always' | 'none' | 'edge' | 'auto' | 'center'`. A sixth mode, `'bounded'`, exists at the implementation level in `ScrollableElement` but is not in the public type.

## The Six Modes

### `auto` (default when no scroll prop)
Shifts the container by one item-width/height per navigation. When `scrollIndex` is set, scrolling doesn't begin until selected reaches `scrollIndex` from the start (or the symmetric position from the end when going backward). Both `auto` and `edge` stop scrolling once the last item is on screen.

### `edge`
Holds position until the NEXT element would go off-screen, then shifts by one item. Uses the renderer's viewport state (`renderState === 8` = InViewPort) to detect visibility. This creates a "lazy scroll" where the list doesn't move until you reach the screen boundary.

### `always`
Places the selected item at the container's starting edge every time. Focus appears to stay fixed at position 0 while the list scrolls behind it.

```ts
nextPosition = -selectedPosition + offset;
```

### `center`
Centers the selected item on screen, clamped to avoid exposing empty space at list start or end.

```ts
const centerPosition = -selectedPosition + (screenSize - selectedSizeScaled) / 2 - screenOffset;
nextPosition = Math.min(Math.max(centerPosition, maxOffset), offset);
```

### `bounded`
A hybrid mode: scrolls normally until the last `upCount` items (default: 6), then freezes. When entering the zone forward, snaps to align the first zone item at the starting edge. Useful for keeping the last few items visible without scrolling them away.

### `none`
No scrolling. The container does not move. Focus shifts between children in place. Note: `scrollToIndex` bypasses this — see [[Row and Column scrollToIndex imperatively sets selected scrolls and focuses without respecting scroll=none]].

## Per-Element Center Override

Any individual child can force center-scroll regardless of container mode by setting `centerScroll=true` on it:
```ts
if (selectedElement.centerScroll) {
  nextPosition = -selectedPosition + (screenSize - selectedSizeScaled) / 2;
}
```

## Internal State Properties

The scroll engine caches state on the container element using underscore-prefixed properties:
- `_initialPosition` — container position at first scroll (set once)
- `_screenOffset` — distance from screen edge to container start (cached on first call)
- `_targetPosition` — last animated target, used to read position mid-animation correctly

These are internal implementation details but they appear on the element, so be aware of naming conflicts.

## Scale Awareness

Scroll calculations account for `selectedElement.scale` and `selectedElement.style?.focus?.scale`, scaling the item's effective size for centering and boundary math.

## onScrolled Callback

```ts
onScrolled?: (elm: ScrollableElement, offset: number, isInitial: boolean) => void;
```
Fires only when position actually changes. `isInitial` is `true` when the new position equals `_initialPosition` (list scrolled back to mount position).

The scroll position change is applied by setting `x` or `y` on the container element, which routes through [[animatable number properties route through transition system before reaching the renderer]]. Since [[Row and Column apply direction-specific transitions on each navigation key press]], the directional transition is merged before `withScrolling` runs, giving forward and backward scrolls different easing curves automatically.

---

Related Entries:
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — Row attaches `scrollRow = withScrolling(true)` on every selection change
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — Column attaches `scrollColumn = withScrolling(false)` on every selection change
- [[VirtualRow and VirtualColumn render a sliding window over large item arrays]] — Virtual components inherit scroll mode from their underlying Row/Column; window shift logic runs alongside withScrolling's position math
- [[bounded scroll mode freezes the list when the last upCount items are in focus]] — extends this by documenting the undocumented bounded scroll variant
- [[edge scroll mode uses renderer viewport state to detect off-screen items before scrolling]] — explains the renderState check used in edge mode

Source: [[primitives-withScrolling]]
Domains:
- [[navigation guide]]
