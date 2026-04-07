---
description: withScrolling uses _screenOffset, _targetPosition, and _initialPosition to track scroll state across calls, avoiding stale reads from animated positions
type: architecture
module: primitives
created: 2026-04-07
---

# withScrolling caches screen offset and target position to handle mid-animation scroll correctly

The scroll engine stores internal state directly on the container element using three underscore-prefixed properties. These exist because CSS-like animations in Lightning TV run asynchronously — reading `el.x` or `el.y` during an animation returns the in-progress animated value, not the target.

## The Three Cached Properties

### `_initialPosition`
Set once on the first scroll call:
```ts
if (componentRef._initialPosition === undefined) {
  componentRef._initialPosition = componentRef[axis];
}
```
Records where the container was at mount. Used by `onScrolled` to report `isInitial=true` when the list returns to its starting position.

### `_screenOffset`
Computed once and cached:
```ts
componentRef._screenOffset = componentRef.offset ?? (isRow ? lng.absX : lng.absY) - componentRef[axis];
```
The gap from the screen edge to the container's starting position. If the container's parent has `clipping=true`, `endOffset` is also computed from the parent's bounds. This is calculated once because it requires reading absolute screen coordinates, which is relatively expensive.

**Gotcha:** If the container's position or the parent layout changes after the first scroll call, `_screenOffset` is stale and scroll calculations will be wrong. This property is never invalidated.

### `_targetPosition`
Updated on every successful scroll:
```ts
componentRef._targetPosition = nextPosition;
```
Used to determine the starting point for the NEXT scroll calculation:
```ts
const targetPosition = componentRef._targetPosition ?? componentRef[axis];
const rootPosition = isIncrementing
  ? Math.min(targetPosition, componentRef[axis])
  : Math.max(targetPosition, componentRef[axis]);
```
This min/max ensures that if the previous animation hasn't finished, the new scroll starts from the correct logical position rather than the mid-animation visual position.

## Naming Conflict Risk

These properties are set directly on the [[ElementNode is the primary abstraction for all renderable elements in Lightning TV Solid]]. They will overwrite any same-named props passed to Row/Column. Avoid prop names `_screenOffset`, `_targetPosition`, and `_initialPosition` on navigable containers.

The stale `_screenOffset` gotcha is particularly relevant because [[animatable number properties route through transition system before reaching the renderer]] — when an animation is in progress, reading `el.x` returns the live animated value, not the committed target. The `_targetPosition` cache exists precisely to avoid using this live value for the next scroll calculation.

---

Related Entries:
- [[withScrolling implements six scroll modes for navigable list containers]] — foundation entry documenting what _screenOffset and _targetPosition are used for across all modes

Source: [[primitives-withScrolling]]
Domains:
- [[navigation guide]]
