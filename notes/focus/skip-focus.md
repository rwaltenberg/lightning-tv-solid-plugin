# skipFocus for Non-Focusable Children

> Non-focusable children (dividers, spacers) need skipFocus={true} so navigable containers skip them during selection.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

If a child within a navigable container should be skipped during navigation (e.g., a divider, decorative element, spacer), set `skipFocus={true}` on it.

Both `navigableForwardFocus` and `spatialForwardFocus` respect `skipFocus`:

- `navigableForwardFocus` reads `navigable.selected`, then finds the first non-`skipFocus` child starting from that index before calling `selectChild()`.
- `spatialForwardFocus` focuses the closest non-`skipFocus` child by screen-space distance.

In `moveSelection()` (the workhorse of list navigation), the scan explicitly skips children with `skipFocus` when searching for the next valid selection. If no valid non-`skipFocus` child is found, `moveSelection` returns `false` to let the event bubble.

## Code Example

```tsx
<view forwardFocus={navigableForwardFocus} selected={0}>
  <Button />
  <view skipFocus={true} /> {/* divider - navigation skips this */}
  <Button />
</view>
```

## Gotchas

- If ALL children have `skipFocus={true}`, `navigableForwardFocus` returns `false`, causing the container itself to become focused (the `forwardFocus` fallback behavior).
- `skipFocus` is checked in `moveSelection()` — if the currently selected index has `skipFocus`, the scan returns `false` and the event bubbles.

## Related Notes

- [focus/forward-focus-required.md] -- navigableForwardFocus and spatialForwardFocus skip skipFocus children
- [api/forward-focus-handlers.md] -- detail on how navigableForwardFocus finds the first non-skipFocus child
- [api/handle-navigation.md] -- moveSelection() scan logic for skipFocus
