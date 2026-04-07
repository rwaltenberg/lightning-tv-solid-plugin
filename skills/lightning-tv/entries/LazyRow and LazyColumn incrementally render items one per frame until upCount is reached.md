---
description: Lazy components expose a growing `offset` slice of `each`, expanding by one item per ~16ms frame until `upCount` items are mounted, then loading more on navigation
type: architecture
module: primitives
created: 2026-04-07
---

`LazyRow` and `LazyColumn` solve the initial render cost problem: mounting hundreds of items at once causes frame drops. The Lazy approach renders items incrementally so the UI is interactive immediately with just a few items visible, and the rest load in the background.

## Required Props

```ts
each: T[]      // Full item array
upCount: number // Items to load before the user scrolls to them
```

`upCount` should be slightly larger than the visible count — enough buffer that the user never sees unrendered items during normal scrolling.

## Loading Phases

### Phase 1: Initial Load (sync: false)

By default, a timer loop fires at ~16ms intervals (one per frame):
```ts
const loadItems = () => {
  if (count < upCount) {
    setOffset(count + 1);
    timeoutId = setTimeout(loadItems, 16);
  }
};
```

This progressively reveals items without blocking the main thread. With `sync: true`, the offset starts at `upCount` and this phase is skipped.

### Phase 2: Navigation-triggered Load

After the initial load, additional items are loaded on demand when the user navigates toward the end:

```ts
// LazyRow: fires on onRight
if (selected >= numChildren - buffer()) {
  setOffset(prev => Math.min(prev + 1, maxOffset));
}
```

Only expands when the user is within `buffer` items of the current render edge.

### Phase 3: Eager Load (eagerLoad: true)

After reaching `upCount`, if `eagerLoad: true`, loading continues using `scheduleTask` (the framework's task queue) until all items are rendered. This is a background pre-load that doesn't block navigation.

## Buffer Calculation

Auto-computed from scroll mode if `buffer` is not explicitly set:
- `auto`, `always`, `bounded`: `upCount + 1`
- `center`: `Math.ceil(upCount / 2) + 1`
- Other: `2`

## lazyScrollToIndex

Expands `offset` to cover the target index before calling the underlying `scrollToIndex`:
```ts
function lazyScrollToIndex(index: number) {
  setOffset(Math.max(index, 0) + buffer());
  queueMicrotask(() => viewRef.scrollToIndex(index));
}
```

## Key Difference from Virtual

Lazy components render a growing prefix of `each` — items are never unloaded. Virtual components render a fixed-size window that shifts — items outside the window are unmounted. Use Lazy for lists where you want items to remain mounted after loading; use Virtual for very large lists where memory is a concern.

Since [[scheduleTask enables deferred background work with high and low priority]], the `eagerLoad: true` mode submits each remaining item as a low-priority task that only runs when the renderer is idle. Since [[task queue suspends on focus change and resumes when renderer is idle]], navigation events automatically pause eager loading and resume it when the user stops moving.

Since [[dollar-prefix state keys in NodeStyles apply style variants based on active states]], items inside LazyRow/LazyColumn typically use `$focus` style variants — these apply through the same state mechanism shared with Row and Column children.

---

Source: [[primitives-Lazy]]
Domains:
- [[performance guide]]
- [[components guide]]
