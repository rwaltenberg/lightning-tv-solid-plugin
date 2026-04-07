---
description: The focusPath array holds ElementNodes from index 0 (currently focused element) up to the last index (root ancestor); order determines capture and bubble traversal direction
type: architecture
module: core
created: 2026-04-07
---

# focus path is an ordered array from focused leaf to root ancestor

The focus manager maintains a module-level `focusPath` array that always reflects the current chain of focused elements. Index `0` is the deepest focused leaf element. The last index is the topmost ancestor in the component tree.

This ordering is not incidental — it directly drives the two-phase event propagation: capture phase iterates from last to 0 (root-to-leaf), and bubble phase iterates from 0 to last (leaf-to-root). Since [[focus manager propagates key events in capture then bubble phase]], the array order is the traversal contract.

The path is rebuilt from scratch on every `setActiveElement` call by walking `elm.parent` links upward from the newly focused element until `parent` is undefined.

```typescript
let current: ElementNode | undefined = currentFocusedElm;
const fp: ElementNode[] = [];
while (current) {
  fp.push(current);
  current = current.parent;
}
```

`focusPath` is exposed externally via `useFocusManager`'s return value as a function `() => focusPath`, giving callers a live view of the current path without exposing the module-level reference. Since [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]], apps can read the live focus path without needing a separate signal subscription.

Row and Column navigation lives inside this path — when a Row child has focus, the path contains the child at index 0, the Row at index 1, and all ancestors above it. The Row's capture handler can intercept `Left`/`Right` before the child's bubble handler fires because [[focus manager propagates key events in capture then bubble phase]] walks this array in opposite directions.

---

Related Entries:
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — rebuilds this array on every focus change by walking parent links

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
