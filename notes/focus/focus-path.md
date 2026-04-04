# Focus Path

> Focus path is an ordered array from leaf to root. $focus state is added/removed as elements enter/leave the path. Common ancestors are unchanged.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

The focus path is the spine of the focus system. It is an ordered array of `ElementNode[]` from the currently focused leaf (index 0) up to the root (last index). Every ancestor in the path is considered "focused."

The focus path is computed in `updateFocusPath()` every time the active element changes:

1. Starting from the newly focused `ElementNode`, walk up via `.parent` to the root.
2. Collect every node along the way into an array (`fp`). Index 0 is the leaf; the last index is the root.
3. For each node in the new path:
   - If it was NOT in the previous focus path (or it is the newly focused leaf), add `Config.focusStateKey` to its `.states`, fire `onFocus`, fire `onFocusChanged(true, ...)`.
4. For each node in the old focus path that is NOT in the new path: remove `Config.focusStateKey` from `.states`, fire `onBlur`, fire `onFocusChanged(false, ...)`.

Ancestor elements that are common to both old and new paths do NOT receive redundant `onFocus`/`onBlur` calls. They remain focused.

The focus state key is the string `'$focus'` (configurable via `Config.focusStateKey`). When an element is in the focus path, this key is added to its `.states` set, enabling style bindings like `$focus: { color: 0xff0000ff }` to react to focus changes automatically.

A reactive SolidJS signal is also available:

```ts
// src/primitives/useFocusManager.ts
export const focusPath: Accessor<ElementNode[]>;
```

The utility functions `isFocused` and `hasFocus` check the `.states` set:

```ts
// src/core/utils.ts
export function isFocused(el: ElementNode | ElementText): boolean;
export const hasFocus: typeof isFocused; // alias
```

## Gotchas

- The focus path array is leaf-first (index 0 = leaf, last index = root). This is the opposite of what DOM `composedPath()` returns.
- Common ancestors between old and new focus paths do NOT receive `onFocus` or `onBlur` — only elements that actually enter or leave the path.
- The `$focus` state key is what drives style bindings — it is added/removed automatically as the path changes.

## Related Notes

- [focus/two-phase-propagation.md] -- focus path is traversed during key event propagation
- [focus/set-active-element-forbidden.md] -- direct setActiveElement bypasses focus path computation
- [focus/forward-focus-required.md] -- forwardFocus determines which element becomes the leaf
- [focus/handler-signature-differences.md] -- onFocus/onBlur/onFocusChanged fire during path updates
