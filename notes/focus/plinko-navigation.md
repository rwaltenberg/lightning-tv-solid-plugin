# Plinko Navigation

> plinko={true} on a Column copies the selected index from the source Row to the target Row when navigating vertically, preserving column position.

**Source**: `spatial-navigation.md` | **Severity**: informational

## Detail

Plinko navigation is used in grids where navigating between rows should maintain the same column (item) position. When `plinko={true}` is set on a navigable element (typically a Column), `moveSelection()` copies the `selected` index from the current child to the target child before focusing it.

This means when moving from Row 0 to Row 1, the `selected` index of Row 0 is applied to Row 1, so focus moves "straight down" to the same column position.

`moveSelection()` behavior with plinko enabled (step 5 in the process):
1. Starting from `el.selected + delta`, scan children in the direction of `delta`.
2. Skip children with `skipFocus`.
3. If `wrap` is enabled, indices wrap around modulo `children.length`.
4. If no valid child is found, return `false`.
5. If `plinko` is enabled, copy the `selected` index from the current child to the target child before focusing it.
6. Call `selectChild()` to apply focus and fire `onSelectedChanged`.

## Code Example

```tsx
<Column plinko={true} forwardFocus={navigableForwardFocus}>
  <Row selected={0} forwardFocus={navigableForwardFocus}>
    <Card /><Card /><Card />
  </Row>
  <Row selected={0} forwardFocus={navigableForwardFocus}>
    <Card /><Card /><Card />
  </Row>
</Column>
```

## Gotchas

- `plinko` copies `selected` index numerically — if Row 0 has 5 items and Row 1 has 3 items, copying index 4 to Row 1 may exceed bounds. `navigableForwardFocus` clamps `selected` to valid range.
- `plinko` is a property on `NavigableProps` and is part of the `NavigableElement` interface.

## Related Notes

- [api/handle-navigation.md] -- moveSelection() detail where plinko is implemented
- [focus/forward-focus-required.md] -- both Column and each Row need forwardFocus
- [focus/skip-focus.md] -- moveSelection() also respects skipFocus alongside plinko
