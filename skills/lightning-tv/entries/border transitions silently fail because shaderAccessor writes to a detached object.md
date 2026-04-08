---
description: Setting transition={{ border: true }} has no visible effect because the shaderAccessor creates a detached target object for animation that the renderer cannot interpolate; borderRadius transitions work because they use a single numeric radius value
type: gotcha
module: core
created: 2026-04-08
---

# border transitions silently fail because shaderAccessor writes to a detached object

When `transition.border` is configured, the `shaderAccessor` setter in `elementNode.ts` follows this path:

1. Detects that `this.lng.shader.props` exists and transition is active
2. Reassigns `target = {}` — a new empty object, **detached** from the live shader props
3. Calls `parseAndAssignShaderProps('border', value, target)`, which writes compound keys like `border-color`, `border-w` into the detached target
4. Calls `this.animate({ shaderProps: target }, animationSettings).start()`

The animation fails silently because:
- The renderer's `animate()` can only interpolate **simple numeric properties**
- Border decomposes into compound keys (`border-color`, `border-w`, `border-gap`, `border-align`) that the animation system does not handle
- The live `this.lng.shader.props` is **never updated** — the new values exist only in the detached target

```tsx
// This will NOT animate — border color stays at 0x00000000
<view
  border={{ width: 2, color: 0x00000000 }}
  transition={{ border: true }}
/>
// Later changing border={{ width: 2, color: 0xff0000ff }} has no visible transition
```

**`borderRadius` transitions DO work** because `borderRadius` maps to a single numeric `radius` key via the `key === 'rounded'` branch, which the renderer can interpolate.

**Workaround:** Instead of transitioning the border, use two layered nodes — an outer node for the border color (using `color`) and an inner node for content. Transition the outer node's `color` property, which is a standard animatable numeric prop.

---

Source: [[core-elementNode]]
Domains:
- [[styling guide]]
- [[core guide]]
