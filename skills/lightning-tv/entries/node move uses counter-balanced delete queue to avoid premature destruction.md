---
description: removeNode pushes -1 and insertNode pushes +1 to a per-node counter; only nodes with a negative total at microtask flush are actually destroyed — allowing atomic moves
type: architecture
module: core
created: 2026-04-07
---

# node move uses counter-balanced delete queue to avoid premature destruction

When SolidJS moves a node (remove from one parent, insert into another), it calls `removeNode` then `insertNode` in the same synchronous task. Without special handling, the remove would destroy the Lightning node before the insert completes.

The delete queue uses a numeric counter on `_queueDelete` to handle this:

```ts
function pushDeleteQueue(node: ElementNode, n: number): void {
  if (node._queueDelete === undefined) {
    node._queueDelete = n;
    if (elementDeleteQueue.push(node) === 1) {
      queueMicrotask(flushDeleteQueue);
    }
  } else {
    node._queueDelete += n;
  }
}
```

- `removeNode` calls `pushDeleteQueue(node, -1)`
- `insertNode` (when `prevParent !== undefined`) calls `pushDeleteQueue(node, 1)`
- Only existing insertions (where the node had a prior parent) get the +1 — brand new insertions skip the queue entirely

At microtask flush:
```ts
function flushDeleteQueue(): void {
  for (let el of elementDeleteQueue) {
    if (Number(el._queueDelete) < 0) {
      el.destroy();
    }
    el._queueDelete = undefined;
  }
}
```

For a move: counter = -1 + 1 = 0, no destruction. For a pure remove: counter = -1, node is destroyed.

The `preserve` property hooks into this same mechanism. Since [[preserve property prevents node destruction by holding the delete counter at zero]], setting `preserve = true` fixes `_queueDelete = 0`, making the node permanently immune to the queue's destruction path.

---

Source: [[solidOpts]]
Domains:
- [[core guide]]
