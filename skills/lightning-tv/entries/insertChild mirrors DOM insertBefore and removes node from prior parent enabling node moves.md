---
description: insertChild() always removes the node from its current parent first (enabling moves), then inserts before the target sibling or appends; mirrors DOM insertBefore semantics
type: api
module: core
created: 2026-04-07
---

# insertChild mirrors DOM insertBefore and removes node from prior parent enabling node moves

`insertChild(node, beforeNode?)` is the internal method SolidJS uses to add children to the element tree. It mirrors the DOM's `insertBefore` semantics:

```typescript
insertChild(node, beforeNode?) {
  // always remove from prior parent first (enables node moves)
  if (node.parent) {
    node.parent.removeChild(node);
    if (!this.rendered) {
      this._hasRenderedChildren = true;
    }
  }

  node.parent = this;

  if (beforeNode) {
    spliceItem(this.children, node, 1);  // remove if already in array
    if (spliceItem(this.children, beforeNode, 0, node) > -1) {
      return;  // inserted before target
    }
  }

  this.children.push(node);  // fallback: append
}
```

Key behaviors:

1. **Automatic removal from prior parent:** If a node already has a parent, `removeChild` is called first. This enables SolidJS to move nodes between containers without manual removal.

2. **`_hasRenderedChildren` flag:** If a node with a rendered `lng` is inserted into an unrendered container, `_hasRenderedChildren = true` is set. During `render()`, this causes the container to re-parent all children's `lng` nodes to the new container's `lng`.

3. **`beforeNode` insertion:** Uses `spliceItem` to find the target and splice the new node in before it. If the target is not found, falls back to append.

4. **SolidJS `<For>` reuse:** SolidJS can reorder existing nodes within a list. Since the same node instance may be moved, this method handles that transparently.

`removeChild` also triggers layout if the parent `requiresLayout()`:
```typescript
removeChild(node) {
  if (spliceItem(this.children, node, 1) > -1) {
    node.onRemove?.call(node, node);
    if (this.requiresLayout()) {
      addToLayoutQueue(this);
    }
  }
}
```

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
