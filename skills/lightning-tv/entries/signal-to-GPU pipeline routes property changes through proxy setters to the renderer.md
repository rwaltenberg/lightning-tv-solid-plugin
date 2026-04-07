---
description: Number props on ElementNode use Object.defineProperty setters that call _sendToLightningAnimatable, routing each write through the transition system before reaching the WebGL renderer
type: architecture
module: core
created: 2026-04-07
---

# signal-to-GPU pipeline routes property changes through proxy setters to the renderer

Every animatable number property on `ElementNode` (`x`, `y`, `w`, `h`, `alpha`, `color`, `rotation`, etc.) is wired via `Object.defineProperty` to a setter that intercepts each write and routes it through the transition/animation system before forwarding to the live renderer node (`this.lng`).

## The Wiring

```typescript
// elementNode.ts — applied for every prop in LightningRendererNumberProps
for (const key of LightningRendererNumberProps) {
  Object.defineProperty(ElementNode.prototype, key, {
    get(): number {
      return this.lng[key];
    },
    set(this: ElementNode, v: number) {
      this._sendToLightningAnimatable(key, v);  // intercept every write
    },
  });
}
```

`LightningRendererNumberProps` includes: `alpha`, `color`, `colorTop/Right/Left/Bottom`, `colorTl/Tr/Bl/Br`, `h`, `fontSize`, `lineHeight`, `mount`, `mountX`, `mountY`, `pivot`, `pivotX`, `pivotY`, `rotation`, `scale`, `scaleX`, `scaleY`, `w`, `worldX`, `worldY`, `x`, `y`, `zIndex`, `zIndexLocked`.

## The Routing Logic

```typescript
// elementNode.ts
_sendToLightningAnimatable(name: string, value: number) {
  if (
    this.transition &&
    this.rendered &&
    Config.animationsEnabled &&
    (this.transition === true ||
      this.transition[name] ||
      this.transition[getPropertyAlias(name)])
  ) {
    const animationSettings =
      this.transition === true || this.transition[name] === true
        ? undefined
        : this.transition[name] || this.transition[getPropertyAlias(name)];

    return (this.lng as any).animateProp(
      name,
      value,
      animationSettings || this.animationSettings || {},
    );
  }

  // No transition — write directly to the renderer node
  (this.lng[name as keyof (IRendererNode | INode)] as number | string) = value;
}
```

Three conditions must ALL be true for a property change to animate:
1. The node has `this.transition` set (either `true` or an object with the key)
2. The node is `this.rendered` (live renderer node exists)
3. `Config.animationsEnabled` is true

If any condition fails, the value is written directly to `this.lng[name]` — synchronous and immediate.

## The Signal Connection

SolidJS signals trigger effects. When a reactive value bound to `node.x` updates, the SolidJS runtime calls the `x` setter on the element node. The setter calls `_sendToLightningAnimatable`, which either calls `animateProp` on the renderer node (producing a GPU-side tween) or writes directly. The GPU renders the result.

This is why transitions "just work" with reactive data — there's no special integration. The proxy setter intercepts every reactive update the same way it intercepts imperative updates.

## Non-Animating Props

A parallel set — `LightningRendererNonAnimatingProps` — uses simpler passthrough setters that write directly to `this.lng[key]` without the transition check. These include `clipping`, `src`, `data`, `fontStretch`, `letterSpacing`, etc.

```typescript
for (const key of LightningRendererNonAnimatingProps) {
  Object.defineProperty(ElementNode.prototype, key, {
    get(): unknown { return this.lng[key]; },
    set(v: unknown) { this.lng[key] = v; },  // direct write, no animation
  });
}
```

---

Related Entries:
- [[animatable number properties route through transition system before reaching the renderer]] — the user-facing description of this same pipeline, from the transition/animation perspective
- [[ElementNode lng property holds a plain object before render and the live renderer node after]] — explains what this.lng is before and after render; writes to lng before render accumulate in a plain object
- [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]] — the direct animate() API that bypasses the setter pipeline entirely

Domains:
- [[core guide]]
