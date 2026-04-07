---
description: Reading selectedNode scans from selected index forward and writes this.selected to the first element node found, even if no navigation occurred
type: gotcha
module: core
created: 2026-04-07
---

# selectedNode getter mutates selected index as a side effect

The `selectedNode` getter on `ElementNode` does not just read — it also writes `this.selected` as a side effect whenever the initial `selected` index points at a non-element node (e.g., a text node injected by SolidJS reconciler).

## Source

```typescript
// elementNode.ts
get selectedNode(): ElementNode | undefined {
  const selectedIndex = this.selected || 0;

  for (let i = selectedIndex; i < this.children.length; i++) {
    const element = this.children[i];
    if (isElementNode(element)) {
      this.selected = i;   // <-- mutation on every call if selectedIndex != i
      return element;
    }
  }

  return undefined;
}
```

The getter starts scanning from `selectedIndex` and advances `i` until it finds an `ElementNode`. If the first child is a text node (or any non-element), the getter silently bumps `this.selected` to the index of the first real element node found.

## When this matters

SolidJS sometimes injects bare text nodes into the children array. If `this.selected` points at one (index 0 often contains a text node when JSX string children are present), reading `selectedNode` will advance `selected` to 1 (the first real ElementNode). Subsequent navigation calculations that rely on `this.selected` will be off by however many non-element nodes were skipped.

```tsx
// Consider a container with mixed children
<view selected={0}>
  {'Label: '}   {/* text node at index 0 */}
  <item />      {/* element at index 1 */}
  <item />      {/* element at index 2 */}
</view>
```

After reading `el.selectedNode`, `el.selected` becomes `1`, not `0`. If `selected` was used as a stable signal elsewhere, this implicit update is surprising.

## Safe usage

- Prefer using `el.children[el.selected]` with an `isElementNode()` guard when you only need to read without side effects.
- Use `selectedNode` only when you also want the corrective mutation behavior.
- Never depend on `selected` remaining stable across a `selectedNode` read when the children array contains non-element nodes.

---

Related Entries:
- [[navigableForwardFocus selects the child at the selected index when a container receives focus]] — also uses the selected index; same implicit skip-over-text-nodes behavior can occur
- [[moveSelection skips skipFocus children and supports plinko index transfer between rows]] — related index-mutation pattern during navigation
- [[node type discriminators allow the flex engine to skip text nodes]] — isElementNode and related type guards used to distinguish element vs text children

Domains:
- [[components guide]]
- [[core guide]]
