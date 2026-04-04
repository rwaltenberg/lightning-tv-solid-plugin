# Node Deletion Queue

> pushDeleteQueue uses a microtask-based counter to distinguish true removal from node reordering.

**Source**: `solidjs-integration.md` | **Severity**: critical

## Detail

SolidJS's universal renderer can move nodes between parents (e.g., during list reordering). To handle this correctly without prematurely destroying GPU resources, `solidOpts.ts` implements a **delete queue** with microtask-based flushing:

- When a node is **inserted** and it already had a parent, `pushDeleteQueue(node, +1)` is called.
- When a node is **removed**, `pushDeleteQueue(node, -1)` is called.
- At the end of the microtask, `flushDeleteQueue` checks each node's `_queueDelete` counter:
  - If the counter is **negative**: the node was truly removed, and `destroy()` is called.
  - If the counter is **zero or positive**: the node was reinserted elsewhere and is preserved.

### The `preserve` Property

The `preserve` property (dynamically defined on `ElementNode.prototype`) provides a boolean interface:
- Setting `preserve = true` sets `_queueDelete = 0`, preventing destruction.

## Code Example

```tsx
// Node moved from one parent to another in the same reactive update:
// 1. SolidJS removes node from parent A --> pushDeleteQueue(node, -1) --> _queueDelete = -1
// 2. SolidJS inserts node into parent B --> pushDeleteQueue(node, +1) --> _queueDelete = 0
// 3. Microtask fires: _queueDelete = 0 --> node is preserved, no destroy() called

// Prevent a node from being destroyed by the queue:
node.preserve = true; // sets _queueDelete = 0
```

## Gotchas

- Without the delete queue, moving a node in list reordering would destroy its GPU resources immediately when it is removed from the first parent.
- The counter approach means rapid insert/remove cycles within a single microtask are handled correctly.
- `preserve = true` is a permanent protection -- it does not expire after one microtask.

## Related Notes

- [core/node-lifecycle.md] -- destroy() called when counter goes negative
- [constraints/create-tag-cleanup.md] -- createTag uses preventCleanup: true, related mechanism
