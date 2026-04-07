---
description: Setting preserve=true fixes _queueDelete to 0, preventing the microtask flush from calling destroy(); preserve=false restores undefined so normal queue processing applies
type: api
module: core
created: 2026-04-07
---

# preserve property prevents node destruction by holding the delete counter at zero

The `preserve` property on `ElementNode` controls whether SolidJS's reconciler can destroy the node when it is removed from the component tree.

It is defined as a computed accessor in `solidOpts.ts`:

```ts
Object.defineProperty(ElementNode.prototype, 'preserve', {
  get(): boolean | undefined {
    return this._queueDelete === 0;
  },
  set(v: boolean) {
    this._queueDelete = v ? 0 : undefined;
  },
});
```

Setting `preserve = true` sets `_queueDelete = 0`. Since `flushDeleteQueue` only destroys nodes where `_queueDelete < 0`, a value of 0 permanently protects the node.

Setting `preserve = false` restores `_queueDelete = undefined`, re-enabling normal delete queue behavior for future removals.

This is used by [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]] to prevent route nodes from being destroyed when navigating away. The preserved nodes remain in the renderer tree but are detached from the SolidJS component tree.

Checking `node.preserve` returns `true` only when `_queueDelete === 0` exactly — nodes in the middle of a move operation (counter = 0 transiently) would also read as preserved, though this is an incidental coincidence since `preserve` is read after the fact.

Since [[node move uses counter-balanced delete queue to avoid premature destruction]] describes the full queue mechanism, `preserve` is best understood as an intentional permanent lock on that counter.

---

Source: [[solidOpts]]
Domains:
- [[core guide]]
- [[performance guide]]
