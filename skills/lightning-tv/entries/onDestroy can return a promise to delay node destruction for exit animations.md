---
description: ElementNode.destroy() checks if onDestroy returns a Promise and waits for resolution before calling _destroy(), enabling await-based exit animation cleanup
type: pattern
module: core
created: 2026-04-07
---

# onDestroy can return a promise to delay node destruction for exit animations

The `destroy()` method inspects the return value of `onDestroy` and handles promises automatically:

```typescript
destroy() {
  if (this.onDestroy) {
    const destroyPromise: unknown = this.onDestroy(this);
    if (destroyPromise instanceof Promise) {
      destroyPromise.then(() => this._destroy());
    } else {
      this._destroy();
    }
  } else {
    this._destroy();
  }
}
```

`_destroy()` calls `this.lng.destroy()` on the underlying renderer node (only if `lng` is an `INode`).

This pattern allows exit animations to complete before the renderer node is freed:

```typescript
// In JSX or component:
onDestroy={async (node) => {
  await node.animate({ alpha: 0 }, { duration: 300 }).start().waitUntilStopped()
  // _destroy() is called after this resolves
}}
```

**Key detail:** The `onDestroy` callback receives the element as both `this` and the first argument. Returning a promise suspends destruction; returning anything else (or nothing) triggers immediate destruction.

Since [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]], the `waitUntilStopped()` method on `IAnimationController` returns a promise, which is exactly what `onDestroy` needs.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
