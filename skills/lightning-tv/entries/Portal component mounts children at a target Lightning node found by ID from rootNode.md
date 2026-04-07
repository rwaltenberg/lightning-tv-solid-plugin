---
description: Renders children at a named Lightning node (found via rootNode.searchChildrenById) or rootNode by default, escaping the current tree position
type: api
module: primitives
created: 2026-04-07
---

# Portal component mounts children at a target Lightning node found by ID from rootNode

`Portal` renders its children at a different location in the Lightning node tree, bypassing z-index and clipping constraints of the current component's ancestors.

```tsx
<Portal>
  <ModalOverlay />
</Portal>

// Mount at a specific named node
<Portal mount="overlay-layer">
  <Toast message="Saved!" />
</Portal>
```

**Mount resolution**: calls `rootNode.searchChildrenById(mount)` — a recursive search from the root. Falls back to `rootNode` if the id is not found or `mount` is omitted.

**Reactive content**: children are wrapped in `createMemo(() => props.children)` via `runWithOwner(owner, ...)` to ensure they run in the portal's reactive scope, not the mount target's scope. The memo is lazily initialized once.

**Cleanup**: when the Portal component unmounts, `onCleanup` sets a `clean` signal to `true`. The `insert` call watches this signal — when true, it calls `dispose()` to remove the inserted content from the target node.

**Returns null** at its own position in the tree — the portal has no visual presence where it appears in JSX.

Common uses: modals, overlays, toast notifications, and focus rings that need to render above all other content regardless of component nesting. Since [[element nodes default color to transparent unless src or explicit color is set]], overlay children that need opacity must explicitly set their color.

---

Source: [[primitives-portal]]
Domains:
- [[components guide]]
