---
description: The lng property is {} during prop accumulation and becomes INode/IRendererNode after render(), enabling safe pre-render prop setting without renderer calls
type: architecture
module: core
created: 2026-04-07
---

# ElementNode lng property holds a plain object before render and the live renderer node after

The `lng` property is the bridge between `ElementNode` and the Lightning renderer. It has two distinct phases:

**Before render:** `lng` is initialized to `{}` (plain object). SolidJS sets props on the element freely — these accumulate in `lng` without triggering any renderer calls. This is safe and cheap.

**After render:** `lng` is replaced with the actual renderer node returned by `renderer.createNode()` or `renderer.createTextNode()`. From this point forward, setting properties on `lng` directly updates the renderer.

```typescript
// Before render - accumulating props
node.lng = {}  // set in constructor
node.lng.x = 100  // just sets a plain object key

// During render()
node.lng = renderer.createNode(props)  // now lng IS the renderer node
node.rendered = true

// After render - live renderer updates
node.lng.x = 200  // calls into renderer directly
```

The `rendered` boolean flag tracks which phase the node is in. Many methods check `this.rendered` before doing live updates (e.g., `_sendToLightningAnimatable`, `setFocus`, `effects` setter).

This design allows SolidJS to apply all JSX props before the node is attached to the scene graph, preventing partial-state renders.

Since [[ElementNode render method controls the two-phase lifecycle from accumulation to live rendering]], the transition happens when a parent-with-rendered-parent is encountered.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
