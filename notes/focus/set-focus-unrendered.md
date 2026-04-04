# setFocus() on Unrendered Elements Sets _autofocus

> Calling setFocus() on an element that has not yet rendered silently sets _autofocus=true and defers. If the element is destroyed before rendering, focus is never transferred.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

`setFocus()` checks whether the element is rendered before proceeding. If the element is not yet rendered:

- It sets `_autofocus = true` on the element (deferred until render).
- It does NOT queue a microtask, fire `onFocus`, or update the active element.
- When the element eventually renders, `_autofocus = true` causes focus to be applied at that point.
- If the element is destroyed before it ever renders, focus is never transferred and is silently lost.

Full `setFocus()` flow:

```
setFocus() called on ElementNode
  |
  +-> Is the element rendered?
  |     NO -> Set _autofocus = true (deferred until render)
  |     YES:
  |       +-> Does it have forwardFocus?
  |       |     ...
  |       +-> Queue a microtask to call setActiveElement(this)
```

The `autofocus={true}` prop is the declarative equivalent and is the recommended approach for initial focus. It also uses `queueMicrotask` to defer the actual focus call, ensuring children have rendered before focus is applied.

## Gotchas

- No error or warning is thrown when `setFocus()` is called on an unrendered element. The failure is silent.
- If the element is destroyed before rendering (e.g., due to a conditional render that flips back), focus is permanently lost.
- Prefer `autofocus={true}` prop for initial focus rather than imperatively calling `setFocus()` before an element is guaranteed to be rendered.

## Related Notes

- [focus/set-focus-microtask.md] -- the microtask queue path for rendered elements
- [focus/forward-focus-required.md] -- forwardFocus is only invoked when the element IS rendered
