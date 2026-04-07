---
description: spatialForwardFocus finds the child whose screen-space center is nearest to the previously active element using halved Euclidean distance, falling back to first focusable child
type: api
module: primitives
created: 2026-04-07
---

# spatialForwardFocus selects the geometrically closest child when a container receives focus

`spatialForwardFocus` is an alternative `ForwardFocusHandler` for navigable containers. Instead of using the stored `selected` index, it calculates which child is visually closest to wherever focus came from.

```ts
export const spatialForwardFocus: ForwardFocusHandler = function () {
  const prevEl = untrack(activeElement);
  if (prevEl) {
    const idx = findClosestFocusableChildIdx(this, prevEl);
    const selected = selectChild(this as NavigableElement, idx);
    if (selected) return true;
  }
  // fallback: first focusable child
  const idx = findFirstFocusableChildIdx(this as NavigableElement);
  return selectChild(this as NavigableElement, idx);
};
```

The distance calculation uses center-to-center Euclidean distance with halved components:
```ts
function distanceBetweenRectCenters(a, b) {
  const dx = Math.abs(a.x + a.width/2  - (b.x + b.width/2))  / 2;
  const dy = Math.abs(a.y + a.height/2 - (b.y + b.height/2)) / 2;
  return Math.sqrt(dx * dx + dy * dy);
}
```
Note: dx and dy are divided by 2 before the Pythagorean step — this is a non-standard weighting that slightly de-emphasizes distance. The relative ranking of children is the same as with standard Euclidean distance, so the "closest" result is identical.

Uses `getElementScreenRect` for absolute screen coordinates, so elements at any scroll position are compared correctly.

**When to use spatialForwardFocus vs navigableForwardFocus:**
- `navigableForwardFocus` — linear lists (Row/Column) where you want to return to the last-selected item
- `spatialForwardFocus` — grid-like containers or wrap layouts where the visually closest item is more intuitive

```tsx
import { spatialForwardFocus } from '@lightningtv/solid/primitives';

<view
  forwardFocus={spatialForwardFocus}
  selected={0}
  onSelectedChanged={(idx, el, child) => { ... }}
>
```

For spatial navigation between items within a container, see [[spatialHandleNavigation navigates flex-wrap containers by finding the closest child in the next row or column]].

---

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
- [[focus guide]]
