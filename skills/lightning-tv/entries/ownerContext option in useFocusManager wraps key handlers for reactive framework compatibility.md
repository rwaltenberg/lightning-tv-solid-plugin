---
description: The ownerContext option accepts a function that wraps keydown and keyup handlers, enabling frameworks like Solid.js to maintain reactive ownership across keyboard events
type: pattern
module: core
created: 2026-04-07
---

# ownerContext option in useFocusManager wraps key handlers for reactive framework compatibility

The `ownerContext` parameter in [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]] is a higher-order wrapper that is called around both the keydown and keyup handlers before they run.

```typescript
const { cleanup } = useFocusManager({
  ownerContext: (cb) => runWithOwner(owner, cb),
});
```

The default value is an identity function that calls `cb()` immediately:
```typescript
ownerContext = (cb) => { cb(); }
```

## Purpose

In Solid.js, reactive computations (signals, memos, effects) require an owner context. DOM event listeners are fired outside the reactive system, so any code inside a keydown handler that reads a signal or creates an effect would be "unowned" — leading to memory leaks or incorrect behavior.

By passing `runWithOwner(owner, cb)` as `ownerContext`, all key-event propagation runs within the correct reactive owner, ensuring effects created during key handling are properly tracked and disposed.

## Scope

`ownerContext` wraps the entire handler invocation — this means all work done by `propagateKeyPress` and `handleKeyEvents` is within the owner context, including calls to element handlers like `onLeft`, `onEnter`, etc.

This is one of the few places in the focus manager where framework-specific concerns leak into core infrastructure — it is deliberately the only such seam. The concrete implementation of this pattern is in [[primitives useFocusManager wraps core focus manager with Solid reactive owner context]], which passes `runWithOwner(owner, ...)` as the `ownerContext`.

---
Source: [[core-focusManager]]
Domains:
- [[focus guide]]
- [[input guide]]
