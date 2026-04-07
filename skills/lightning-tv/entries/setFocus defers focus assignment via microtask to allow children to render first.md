---
description: setFocus() queues the actual focus change via queueMicrotask so children (Row, Column) finish rendering before focus is applied; last caller in a sync batch wins
type: api
module: core
created: 2026-04-07
---

# setFocus defers focus assignment via microtask to allow children to render first

`setFocus()` does not immediately set focus. It uses a module-level deduplication pattern to defer the actual `setActiveElement()` call until the next microtask.

```typescript
setFocus(): void {
  if (this.rendered) {
    if (this.forwardFocus !== undefined) {
      // ... handle forwarding
    }
    nextActiveElement = this;          // module-level variable
    if (focusQueued === false) {
      focusQueued = true;
      queueMicrotask(() => {
        focusQueued = false;
        if (nextActiveElement) {
          const element = nextActiveElement;
          nextActiveElement = null;
          setActiveElement(element);
        }
      });
    }
  } else {
    this._autofocus = true;  // deferred until render()
  }
}
```

**Deduplication:** `focusQueued` prevents multiple microtasks from being queued. Only one microtask runs per sync batch. The last `setFocus()` call within the sync batch sets `nextActiveElement`, so it wins.

**Pre-render path:** If the node is not rendered, `_autofocus = true` is set instead. When `render()` completes, it checks `node._autofocus && node.setFocus()`.

**forwardFocus handling:**
- If `forwardFocus` is a function: calls it with `(this)`. If it returns `false`, focus falls back to this node (otherwise the function is responsible for focusing a child)
- If `forwardFocus` is a number: focuses the child at that index

The microtask deferral is particularly important for [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]], which forward focus to their selected child — the child must be rendered before focus can propagate to it. Since [[navigableForwardFocus selects the child at the selected index when a container receives focus]] calls `setFocus()` on the selected child, and that child might not yet be rendered when a navigation component mounts, the microtask deferral guarantees safe ordering.

---

Related Entries:
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — the function called after the microtask resolves, which actually commits the focus change

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[focus guide]]
