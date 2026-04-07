---
description: emit(event, ...args) walks the parent chain calling onX handlers; stops and returns true if any handler returns true, enabling event cancellation
type: api
module: core
created: 2026-04-07
---

# ElementNode emit method bubbles custom events up the parent chain

`emit(event, ...args)` implements event bubbling through the element tree. It converts the event name to `onX` format and walks up the parent chain.

```typescript
emit(event: string, ...args: any[]): boolean {
  let current = this as ElementNode;
  const capitalizedEvent = `on${event.charAt(0).toUpperCase()}${event.slice(1)}`;

  while (current) {
    const handler = current[capitalizedEvent];
    if (isFunction(handler)) {
      if (handler.call(current, this, ...args) === true) {
        return true;
      }
    }
    current = current.parent!;
  }
  return false;
}
```

Key details:
- Event name `'select'` becomes `'onSelect'`; `'loaded'` becomes `'onLoaded'`
- Handler is called with `this` bound to the **current node in the chain** (where the handler is defined), not the originating node
- First argument is always the **original node** that emitted the event
- Returning exactly `true` from a handler stops propagation and causes `emit` to return `true`
- Returning anything else (including `undefined`) continues bubbling
- Returns `false` if no handler stopped propagation

This is distinct from `onEvent` (renderer events like `loaded`, `failed`, `inBounds`) — `emit` is for custom application-level events.

Usage:
```typescript
// Emit from a child
childNode.emit('select', { index: 2 })
// Handler on a parent:
// onSelect(originalNode, { index }) { ... return true /* stops bubble */ }
```

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[focus guide]]
