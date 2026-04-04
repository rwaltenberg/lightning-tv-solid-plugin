# setFocus() Uses queueMicrotask — Multiple Calls Coalesced

> setFocus() queues focus via queueMicrotask. Multiple synchronous calls coalesce and only the last one wins.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

When `setFocus()` is called on a rendered element without `forwardFocus`, it does not immediately set focus. Instead it queues a microtask to call `setActiveElement(this)`. The coalescing is implemented via a `nextActiveElement`/`focusQueued` variable: if `setFocus()` is called multiple times synchronously, only the last call's element is stored in `nextActiveElement`, and when the microtask executes it applies that single value.

Full `setFocus()` flow:

```
setFocus() called on ElementNode
  |
  +-> Is the element rendered?
  |     NO -> Set _autofocus = true (deferred until render)
  |     YES:
  |       +-> Does it have forwardFocus?
  |       |     YES (function) -> Call forwardFocus(). If result !== false, return.
  |       |     YES (number)   -> Call children[n].setFocus(), return.
  |       |     NO:
  |       +-> Queue a microtask to call setActiveElement(this)
  |           (Coalesced: only the LAST setFocus() in a microtask batch wins)
```

The `autofocus={true}` prop also uses `queueMicrotask` to defer the actual focus call, ensuring children have rendered before focus is applied.

## Gotchas

- If `setFocus()` is called on element A then immediately on element B in the same synchronous block, only element B receives focus. Element A's `onFocus` callback never fires.
- This coalescing is intentional for performance (e.g., during a `<For>` render loop where many children call `setFocus()`), but can be confusing during debugging.
- The microtask coalescing only applies when both calls happen before the current microtask queue drains. Asynchronous calls separated by awaits are not coalesced.

## Related Notes

- [focus/set-focus-unrendered.md] -- behavior when setFocus() is called on an unrendered element
- [focus/forward-focus-required.md] -- forwardFocus intercepts setFocus() before the microtask queue
- [focus/set-active-element-forbidden.md] -- why calling setActiveElement directly bypasses this coalescing
