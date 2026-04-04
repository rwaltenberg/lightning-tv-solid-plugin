# forwardFocus Required on Container Elements

> Containers MUST declare forwardFocus or setFocus() claims focus on the container itself instead of delegating to children.

**Source**: `spatial-navigation.md` | **Severity**: critical

## Detail

If a container element (one that holds focusable children) does not declare `forwardFocus`, then `setFocus()` on it will make the container itself the active element. Key handlers on children will never fire because the children are not in the focus path. This is described as the single most common mistake.

`forwardFocus` is a property on `ElementNode` that is either a `number` index or a `ForwardFocusHandler` function:

- If it is a **function**: Called with `this` bound to the element. If it returns anything other than `false`, focus delegation is considered handled and `setFocus()` returns. If it returns `false`, the element takes focus itself.
- If it is a **number**: The child at that index receives `setFocus()` instead.

`forwardFocus` chains can be nested: a root container can forward to a child container, which in turn forwards to its own child. The chain terminates when an element without `forwardFocus` is reached, and that element becomes the leaf focus target.

If `forwardFocus` returns `false` (e.g., the container has no children), `setFocus()` falls through and the container itself becomes focused. Ensure containers always have at least one focusable child, or handle the empty-children case explicitly.

## Code Example

```tsx
// Standard list navigation (Row/Column)
<view
  selected={0}
  forwardFocus={navigableForwardFocus}
  onLeft={handleNavigation('left')}
  onRight={handleNavigation('right')}
  onSelectedChanged={(idx, el, child, lastIdx) => { /* scroll logic, etc. */ }}
>
  {items.map(item => <Card />)}
</view>

// Spatial/grid navigation in flex-wrap layouts
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

- Without `forwardFocus`, the container absorbs focus and children's key handlers never fire.
- If `forwardFocus` returns `false`, the container itself becomes the active element — this happens when there are no focusable children.
- `forwardFocus` chains: each container in a hierarchy must declare its own `forwardFocus`. Missing it at any level in the chain breaks delegation.
- For `autofocus={true}`, the framework uses `queueMicrotask` to defer the actual focus call, ensuring children have rendered first before `forwardFocus` is invoked.

## Related Notes

- [focus/use-focus-manager-once.md] -- useFocusManager must be initialized before forwardFocus is exercised
- [focus/set-focus-microtask.md] -- setFocus() uses queueMicrotask, relevant to how forwardFocus chains resolve
- [api/forward-focus-handlers.md] -- full API for navigableForwardFocus and spatialForwardFocus
- [focus/skip-focus.md] -- navigableForwardFocus skips children with skipFocus={true}
