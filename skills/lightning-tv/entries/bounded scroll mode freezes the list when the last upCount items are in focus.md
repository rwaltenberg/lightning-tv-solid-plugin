---
description: The bounded scroll mode (implementation-only, not in NavigableProps type) stops list movement when focus enters the last upCount items, creating a fixed end zone
type: gotcha
module: primitives
created: 2026-04-07
---

# bounded scroll mode freezes the list when the last upCount items are in focus

`bounded` is a scroll mode that combines scrolling with a frozen end zone. It is present in the `ScrollableElement` interface used by `withScrolling`, but it is NOT exposed in the public `NavigableProps.scroll` type. To use it, set the prop directly on the element node (bypassing TypeScript) or cast.

```ts
// NavigableProps.scroll type: 'always' | 'none' | 'edge' | 'auto' | 'center'
// ScrollableElement.scroll type: adds 'bounded'
```

**Behavior:** The list scrolls normally until focus reaches item `totalItems - upCount`. At that point, scrolling freezes. The last `upCount` items become visible simultaneously as the scroll stops.

```ts
const nonScrollableZoneStart = Math.max(0, totalItems - upCount);
const isInNonScrollableZone = selected >= nonScrollableZoneStart;
```

`upCount` defaults to 6 if not set on the element.

**Entry into the zone:** When moving forward into the zone (specifically when `selected === nonScrollableZoneStart` for the first time), the list snaps to align the first zone item at the starting edge, then freezes.

**Within the zone:** Position is frozen (`rootPosition`). Focus moves between the last items without list movement.

**Exiting the zone backward:** When moving back from `nonScrollableZoneStart`, normal scrolling resumes.

The practical use case is TV channel lists or menus where you want the final group of items to remain visible together, not scroll one-by-one off screen.

Since [[withScrolling implements six scroll modes for navigable list containers]], `bounded` participates in the same position-clamping infrastructure as other modes, with one exception: the final bounds clamp is disabled for `bounded` mode (`scroll !== 'bounded'` in the clamp condition).

---

Source: [[primitives-withScrolling]]
Domains:
- [[navigation guide]]
