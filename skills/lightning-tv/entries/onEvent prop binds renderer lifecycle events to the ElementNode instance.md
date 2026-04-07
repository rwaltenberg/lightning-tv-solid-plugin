---
description: onEvent is an object of handlers for renderer events (loaded, failed, freed, inBounds, outOfBounds, inViewport, outOfViewport); bound during render() with node as both this and first argument
type: api
module: core
created: 2026-04-07
---

# onEvent prop binds renderer lifecycle events to the ElementNode instance

The `onEvent` property accepts an object mapping renderer event names to handlers. These are bound during `render()` after the Lightning renderer node is created:

```typescript
if (node.onEvent) {
  for (const [name, handler] of Object.entries(node.onEvent)) {
    if (typeof node.lng.on === 'function') {
      node.lng.on(name, (_inode, data) => handler.call(node, node, data));
    }
  }
}
```

Available events:
- `'loaded'` — texture/image finished loading
- `'failed'` — texture/image failed to load
- `'freed'` — node freed from GPU memory
- `'inBounds'` — node entered renderer bounds
- `'outOfBounds'` — node left renderer bounds
- `'inViewport'` — node entered the viewport
- `'outOfViewport'` — node left the viewport

Handler signature: `(node: ElementNode, data: any) => void`

`this` is bound to the `ElementNode` instance (not the renderer INode). The original renderer `_inode` argument is discarded — you always receive the Solid-side `ElementNode` as the first argument.

Usage:
```typescript
const onEvent: OnEvent = {
  loaded: (node, data) => { node.autosize && node.parent?.updateLayout() },
  outOfBounds: (node) => { /* free resources */ },
}
```

**Key detail:** `onEvent` is bound once during render. If you need to add/remove event listeners dynamically, you must access `node.lng.on()` directly.

**Distinction from `emit()`:** `onEvent` is for renderer-internal events (texture lifecycle, viewport). `emit()` is for custom application events bubbling through the tree.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
