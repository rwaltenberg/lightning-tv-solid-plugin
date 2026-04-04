# Forward Focus Handlers API

> navigableForwardFocus behavior (reads selected, finds non-skipFocus child). spatialForwardFocus behavior (screen-space distance). ForwardFocusHandler signature.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

Located in `src/primitives/utils/handleNavigation.ts`.

**`ForwardFocusHandler` signature:**

```ts
type ForwardFocusHandler = (
  this: ElementNode,
  elm: ElementNode,
) => boolean | void;
```

Set on `ElementNode.forwardFocus`. When `setFocus()` is called on an element with `forwardFocus`:
- If it is a **function**: Called with `this` bound to the element. If it returns anything other than `false`, focus delegation is considered handled and `setFocus()` returns. If it returns `false`, the element takes focus itself.
- If it is a **number**: The child at that index receives `setFocus()` instead.

**`navigableForwardFocus`:**

```ts
export const navigableForwardFocus: ForwardFocusHandler;
```

Behavior:
1. Reads `navigable.selected` (clamped to valid range).
2. Finds the first non-`skipFocus` child starting from the `selected` index.
3. Calls `selectChild()`, which sets `selected`, calls `child.setFocus()`, and fires `onSelectedChanged`.

If `forwardFocus` returns `false` (e.g., no focusable children), `setFocus()` falls through and the container itself becomes focused.

**`spatialForwardFocus`:**

```ts
export const spatialForwardFocus: ForwardFocusHandler;
```

Behavior:
1. Gets the previously focused element via the `activeElement` signal (untracked).
2. Computes the screen-space center-to-center distance from the previous element to each child.
3. Focuses the closest non-`skipFocus` child.
4. Falls back to `findFirstFocusableChildIdx` if no previous element exists.

**Deprecated:**

```ts
/** @deprecated Use navigableForwardFocus instead */
export function onGridFocus(cb?: OnSelectedChanged): ForwardFocusHandler;
```

## Code Example

```tsx
// Standard list navigation
<view
  selected={0}
  forwardFocus={navigableForwardFocus}
  onLeft={handleNavigation('left')}
  onRight={handleNavigation('right')}
>
  {items.map(item => <Card />)}
</view>

// Spatial/grid navigation
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

- If `navigableForwardFocus` returns `false` (no focusable children), the container itself becomes focused — this is by design.
- `spatialForwardFocus` requires elements to be rendered with valid screen positions for distance calculation to work.
- `spatialForwardFocus` reads `activeElement` untracked — it uses the screen position of the previously focused element at the moment of focus entry.
- `onGridFocus` is deprecated; use `navigableForwardFocus` instead.

## Related Notes

- [focus/forward-focus-required.md] -- forwardFocus must be set on all containers
- [focus/skip-focus.md] -- both handlers skip children with skipFocus
- [focus/spatial-navigation.md] -- usage guide for spatialForwardFocus
- [api/handle-navigation.md] -- key handlers that pair with these forward focus handlers
