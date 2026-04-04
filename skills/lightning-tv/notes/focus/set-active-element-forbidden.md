# Never Call setActiveElement Directly

> setActiveElement from focusManager.ts and activeElement.ts are two different functions with different roles. Calling either directly bypasses the forwardFocus chain and microtask coalescing.

**Source**: `spatial-navigation.md` | **Severity**: critical

## Detail

There are two different `setActiveElement` functions in different files:

- `src/core/focusManager.ts`: Updates the focus path and calls `Config.setActiveElement`. This is the core engine's internal setter.
- `src/activeElement.ts`: A SolidJS signal setter (`Setter<ElementNode | undefined>`). This only updates the reactive signal.

Both are exported:

```ts
// src/activeElement.ts
export const activeElement: Accessor<ElementNode | undefined>;
export const setActiveElement: Setter<ElementNode | undefined>;
```

Calling either directly:
- Bypasses the `forwardFocus` chain (no delegation to children occurs).
- Bypasses microtask coalescing (no `queueMicrotask` batching).
- Calling the SolidJS signal setter directly does not update the focus path or fire `onFocus`/`onBlur` callbacks.
- Calling the core setter directly skips `forwardFocus` resolution.

The only correct way to imperatively move focus is `element.setFocus()`.

## Gotchas

- There are two `setActiveElement` exports in the codebase from different files — they are not the same function and must not be used interchangeably.
- Direct calls will not fire `onFocus`, `onBlur`, or `onFocusChanged` callbacks because the focus path is not updated.
- Direct calls will not respect `forwardFocus`, meaning containers will steal focus instead of delegating.

## Related Notes

- [focus/set-focus-microtask.md] -- setFocus() is the correct imperative API, uses queueMicrotask
- [focus/forward-focus-required.md] -- forwardFocus chain that setFocus() respects
- [focus/focus-path.md] -- focus path computation that direct setActiveElement calls skip
