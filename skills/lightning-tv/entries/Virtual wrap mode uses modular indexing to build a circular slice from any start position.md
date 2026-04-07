---
description: When `wrap: true`, the slice is built using `mod(start + i, total)` so the array wraps around; combined with a -1 item offset, circular navigation appears seamless
type: pattern
module: primitives
created: 2026-04-07
---

VirtualRow and VirtualColumn support circular navigation via `wrap: true`. The implementation uses modular arithmetic to build a slice that wraps around the array boundaries.

## Slice Construction

```ts
newSlice = props.wrap
  ? Array.from(
      { length },
      (_, i) => items()[utils.mod(start + i, total)],
    )
  : items().slice(start, start + length);
```

`utils.mod` (unlike `%`) handles negative numbers correctly, so `mod(-1, total) === total - 1`. This means the slice can start "before" index 0 and correctly pull from the end of the array.

## Initial Offset

At initialization with `wrap: true`, the container is offset by `-1 * childSize`:

```ts
viewRef.lng[axis] = (viewRef.lng[axis] || 0) + childSize * -1;
originalPosition = viewRef.lng[axis];
```

This ensures there's always one item visually before the focused item. Without this offset, wrapping backward from item 0 would appear to jump forward in the list before animating.

## Delta Normalization

For wrap mode, navigation deltas are normalized to stay within the window:

```ts
function normalizeDeltaForWindow(delta: number, windowLen: number): number {
  const half = windowLen / 2;
  if (delta > half) return delta - windowLen;
  if (delta < -half) return delta + windowLen;
  return delta;
}
```

This prevents a jump from the last item to the first from being treated as a huge positive delta when it should be a small negative delta (-1).

## Supported Scroll Modes with Wrap

All scroll modes support wrap except `'none'`. The modes differ in where `selected` pins within the wrap window:
- `'auto'`: pins at `scrollIndex`
- `'always'`: pins at index 1
- `'edge'`: selection moves until it reaches `displaySize`, then window shifts

---

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
