# Spatial Navigation

> spatialForwardFocus and spatialHandleNavigation enable flex-wrap grid navigation using screen-space distance to select the closest child.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

Spatial navigation is used for flex-wrap containers where children are laid out in a grid without fixed rows/columns. It uses two complementary utilities:

- `spatialForwardFocus` (`ForwardFocusHandler`): Used on `forwardFocus`. Selects the child to focus when entering the container.
- `spatialHandleNavigation` (`KeyHandler`): Used on directional handlers. Moves focus within the container.

**`spatialForwardFocus` behavior:**
1. Gets the previously focused element via the `activeElement` signal (untracked).
2. Computes the screen-space center-to-center distance from the previous element to each child.
3. Focuses the closest non-`skipFocus` child.
4. Falls back to `findFirstFocusableChildIdx` if no previous element exists.

**`spatialHandleNavigation` behavior:**
1. Determines flex direction (`flexDirection === 'column'` -> primary axis is `y`; default is `x`).
2. Maps the arrow key to movement along the flex axis or cross axis.
3. Flex-axis movement: Scans forward/backward among siblings that share the same cross-axis position (same row/column).
4. Cross-axis movement: Finds the closest child in the next row/column by measuring distance along the flex axis from the current child's position.

## Code Example

```tsx
<view
  selected={0}
  display="flex"
  flexWrap="wrap"
  forwardFocus={spatialForwardFocus}
  onUp={spatialHandleNavigation}
  onDown={spatialHandleNavigation}
  onLeft={spatialHandleNavigation}
  onRight={spatialHandleNavigation}
>
  {tiles.map(tile => <Tile />)}
</view>
```

## Gotchas

- `spatialForwardFocus` uses screen-space distance (center-to-center) not index proximity — it requires elements to be rendered with valid positions.
- If no previous element exists (e.g., first time entering the container), it falls back to `findFirstFocusableChildIdx`.
- Children with `skipFocus={true}` are excluded from both distance calculations and fallback selection.

## Related Notes

- [focus/forward-focus-required.md] -- spatialForwardFocus is a ForwardFocusHandler assigned to forwardFocus
- [focus/skip-focus.md] -- spatialForwardFocus skips children with skipFocus
- [api/forward-focus-handlers.md] -- full API detail for spatialForwardFocus
- [api/handle-navigation.md] -- full API detail for spatialHandleNavigation
