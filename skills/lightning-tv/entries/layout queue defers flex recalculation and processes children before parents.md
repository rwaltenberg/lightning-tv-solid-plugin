---
description: A module-level Set and boolean flag batch layout runs via reprocessUpdates or queueMicrotask; runLayout processes the queue in reverse (index high to low) to ensure children before parents
type: architecture
module: core
created: 2026-04-07
---

# layout queue defers flex recalculation and processes children before parents

Flex layout is never calculated synchronously when a property changes. Instead, nodes that need layout are added to a `Set<ElementNode>` and processed asynchronously.

## Queue mechanism

```typescript
const layoutQueue = new Set<ElementNode>();
let layoutRunQueued = false;

function addToLayoutQueue(node: ElementNode) {
  layoutQueue.add(node);
  if (!layoutRunQueued) {
    layoutRunQueued = true;
    if ('reprocessUpdates' in renderer.stage && renderer.stage.reprocessUpdates) {
      renderer.stage.reprocessUpdates(runLayout);
    } else {
      queueMicrotask(runLayout);
    }
  }
}
```

The `Set` deduplicates — adding the same node multiple times is safe and cheap.

`reprocessUpdates` is preferred over `queueMicrotask` when available on the renderer stage, likely to integrate with the renderer's own update cycle.

## Processing order

```typescript
function runLayout() {
  while (layoutQueue.size > 0) {
    const queue = [...layoutQueue];
    layoutQueue.clear();
    for (let i = queue.length - 1; i >= 0; i--) {
      const node = queue[i] as ElementNode;
      node.updateLayout();
    }
  }
  layoutRunQueued = false;
}
```

The `for` loop runs **in reverse** — highest index first. Since children are inserted before parents in the tree-building process, this ensures children are laid out before parents. The `while` loop handles the case where `updateLayout()` adds more nodes to the queue (e.g., when a flex child change causes the parent to need re-layout).

## When nodes are added

- When a child is removed from a node that `requiresLayout()`
- When `render()` is called on a child of a flex container
- When `updateLayout()` propagates a change to the parent
- After text images `loaded` events if parent requires layout
- On `topNode` initial render

Since [[ElementNode updateLayout method runs flex and onLayout for flex containers]], `updateLayout` is the actual computation step that this queue defers.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[performance guide]]
