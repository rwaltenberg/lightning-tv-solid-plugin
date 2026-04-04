# Chain While Running Resets Queue

> Calling chain() on a running animation clears _animationQueue and resets _animationRunning. Build the entire chain before calling start().

**Source**: `core-rendering-nodes.md` | **Severity**: important

## Detail

When `chain()` is called on a node that is already running an animation chain, the following happens:

```ts
if (this._animationRunning) {
  this._animationQueue = [];
  this._animationRunning = false;
}
```

This is an **implicit cancellation**, not a merge. The existing queue is cleared and the running flag is reset. The new `chain()` call then starts a fresh queue.

This means: if `start()` has been called and the chain is executing, calling `chain()` again will silently discard all remaining queued animations and any animations currently in-flight from the previous `start()`.

### Correct Pattern

Build the entire chain **before** calling `start()`:

```ts
// CORRECT -- full chain built before start()
nodeRef
  .chain({ x: 100 }, { duration: 300 })
  .chain({ y: 200 })
  .chain({ alpha: 0 })
  .start();  // returns Promise<void>
```

### How chain() Works

Each `chain()` call pushes to `_animationQueue`. `start()` pops and awaits each animation sequentially via `waitUntilStopped()`.

## Code Example

```ts
// CORRECT -- build entire chain then start
nodeRef
  .chain({ x: 100 }, { duration: 300 })
  .chain({ y: 200 }, { duration: 200 })
  .chain({ alpha: 0 }, { duration: 150 })
  .start();

// WRONG -- calling chain() after start() clears the running chain
nodeRef
  .chain({ x: 100 })
  .start();                        // starts executing
// ...later in same frame:
nodeRef.chain({ y: 200 });         // CLEARS the running chain!
```

## Gotchas

- `chain()` returns `this` (the `ElementNode`) for method chaining -- do not confuse it with `animate()` which returns an `IAnimationController`.
- `start()` returns `Promise<void>` -- awaiting it means waiting for ALL chained animations to complete.
- Calling `chain()` while running is a silent destructive operation with no warning in dev or prod.

## Related Notes

- [constraints/animate-before-render.md] -- animate() also requires rendered === true
- [api/element-node.md] -- animate(), chain(), start() method signatures
