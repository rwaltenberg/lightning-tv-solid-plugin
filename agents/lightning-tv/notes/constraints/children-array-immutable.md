# Children Array Immutable

> The children array is managed by insertChild/removeChild. Direct mutation bypasses layout queue, parent tracking, and onRemove callbacks.

**Source**: `core-rendering-nodes.md` | **Severity**: critical

## Detail

The `children` array on `ElementNode` is managed exclusively through `insertChild` and `removeChild`. These methods maintain:

- **Parent tracking**: `node.parent = parent` (or `null` on removal).
- **Layout queue**: Updates to the layout queue when children are added/removed from flex containers.
- **`onRemove` callback**: Fires `onRemove(node)` when a child is removed.
- **Position tracking**: `insertChild` follows DOM `insertBefore` semantics. If a node already has a parent, it is removed from the old parent first.

### What insertChild Does

```
parent.insertChild(node, beforeNode?)
  |
  node.parent = parent
  if (beforeNode) --> splice into correct position
  else --> push to children array
  if (node had previous parent) --> removeChild from old parent
```

### What Direct Mutation Bypasses

Directly mutating `node.children` (e.g., `node.children.push(child)`, `node.children.splice(...)`) bypasses all of the above -- no parent tracking, no layout queue updates, no `onRemove` callbacks.

## Gotchas

- SolidJS manages child insertion/removal automatically through `insertNode` and `removeNode` in `solidOpts.ts` -- this anti-pattern is only a risk if you manually manipulate the tree.
- If you must traverse children, read `node.children` but never mutate it directly.
- `getChildById` and `searchChildrenById` are the correct ways to find children.

## Related Notes

- [core/node-lifecycle.md] -- insertChild in Phase 3 (tree insertion)
- [core/node-deletion-queue.md] -- deletion queue that wraps removeChild
- [api/element-node.md] -- insertChild, removeChild, getChildById API
