---
description: chain() queues animation steps that execute sequentially when start() is called; calling chain() while an animation runs resets the queue and starts fresh
type: api
module: core
created: 2026-04-07
---

# ElementNode chain and start methods enable sequenced animation queuing

`chain()` and `start()` together implement a sequential animation queue. Each `chain()` call adds a step to `_animationQueue`, and `start()` runs them one at a time using `await`.

## `chain(props, animationSettings?)` — returns `this`

```typescript
chain(props, animationSettings?) {
  if (this._animationRunning) {
    // Reset queue if called mid-animation
    this._animationQueue = [];
    this._animationRunning = false;
  }
  // settings: per-call > first call's settings > node defaults
  animationSettings = animationSettings || this._animationQueueSettings;
  this._animationQueue = this._animationQueue || [];
  this._animationQueue.push({ props, animationSettings });
  return this;
}
```

The settings priority: first explicit `animationSettings` in the chain is stored as `_animationQueueSettings` and used for subsequent steps that don't specify their own.

## `start()` — async, returns void

```typescript
async start() {
  let animation = this._animationQueue!.shift();
  while (animation) {
    this._animationRunning = true;
    await this.animate(animation.props, animation.animationSettings)
      .start()
      .waitUntilStopped();
    animation = this._animationQueue!.shift();
  }
  this._animationRunning = false;
  this._animationQueueSettings = undefined;
}
```

Each step calls `animate().start().waitUntilStopped()` — the animation runs to completion before the next step begins.

**Gotcha:** If `chain()` is called while `_animationRunning === true`, the queue is wiped and the running animation continues to its end before the new queue begins. The new chain effectively replaces any pending steps.

Usage pattern:
```typescript
node
  .chain({ x: 100 }, { duration: 200 })
  .chain({ alpha: 0 })  // uses same duration from first call
  .start()
```

Since [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]], `start()` internally calls `animate()` which requires rendered state.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
