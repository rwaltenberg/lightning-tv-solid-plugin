---
description: `scrollToIndex` restores `this.lng[axis]` to `originalPosition` before computing the new target, preventing cumulative animation offsets from misaligning the jump
type: gotcha
module: primitives
created: 2026-04-07
---

When Virtual components shift the rendered window, they animate the container element by modifying its renderer position (`this.lng.x` or `this.lng.y`). After multiple navigations, this position drifts from the Solid-side `x`/`y` props. If `scrollToIndex` then fires without accounting for this drift, the jump lands at the wrong visual position.

## The Fix

`originalPosition` captures the container's position before any animation has occurred:

```ts
originalPosition = originalPosition ?? elm[axis];
```

On `scrollToIndex`:
```ts
if (originalPosition !== undefined) {
  viewRef.lng[axis] = originalPosition;
  targetPosition = originalPosition;
}
```

By resetting to `originalPosition` first, the jump is computed from a clean baseline regardless of how many window shifts (and their accumulated offsets) occurred before.

## Why originalPosition Uses ??=

`originalPosition` is set to the element's position at first focus (`onSelectedChanged` first call). It uses `??=` so it's only captured once — before any animation has drifted the position. Subsequent `onSelectedChanged` calls that check `originalPosition` always see the pre-animation baseline.

## Wrap Mode Offset

For `wrap: true`, the container is intentionally offset by `-1 * childSize` at initialization. The `originalPosition` captured after this offset becomes the new baseline, so `scrollToIndex` jumps to the correct wrap-aware position.

---

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
