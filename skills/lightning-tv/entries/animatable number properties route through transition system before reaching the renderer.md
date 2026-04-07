---
description: LightningRendererNumberProps (x, y, w, h, alpha, color, rotation, scale, etc.) check the transition config and call animateProp() instead of setting directly when a transition is configured
type: architecture
module: core
created: 2026-04-07
---

# animatable number properties route through transition system before reaching the renderer

All numeric properties that can be animated are defined in `LightningRendererNumberProps` and proxied through `_sendToLightningAnimatable()` on set. This is how the `transition` prop enables declarative animation.

```typescript
_sendToLightningAnimatable(name: string, value: number) {
  if (
    this.transition &&
    this.rendered &&
    Config.animationsEnabled &&
    (this.transition === true || this.transition[name] || this.transition[getPropertyAlias(name)])
  ) {
    // ... resolve animation settings
    return (this.lng as any).animateProp(name, value, animationSettings);
  }
  // fall through: set directly
  (this.lng[name] as number | string) = value;
}
```

Conditions for animation to occur:
1. `this.transition` is set (truthy)
2. `this.rendered` is true (must be rendered first)
3. `Config.animationsEnabled` is true
4. The specific property name (or its alias — `w`↔`width`, `h`↔`height`) is in the transition config

The full list of animatable number props: `alpha`, `color`, `colorTop`, `colorRight`, `colorLeft`, `colorBottom`, `colorTl`, `colorTr`, `colorBl`, `colorBr`, `h`, `fontSize`, `lineHeight`, `mount`, `mountX`, `mountY`, `pivot`, `pivotX`, `pivotY`, `rotation`, `scale`, `scaleX`, `scaleY`, `w`, `worldX`, `worldY`, `x`, `y`, `zIndex`, `zIndexLocked`.

Non-animatable props (text props, clipping, texture, etc.) bypass this system and set `lng` directly — see `LightningRendererNonAnimatingProps`.

**Gotcha:** Setting `x` or `y` before `rendered` is true always sets directly (no transition), even if `transition` is configured. Transitions only apply post-render.

---

Related Entries:
- [[FadeInOut component blocks node destruction to play exit animation before removal]] — uses the alpha animatable property to produce enter/exit transitions
- [[Row and Column apply direction-specific transitions on each navigation key press]] — merges x/y transition configs on each keypress to animate the container position
- [[NavigableProps transitionLeft right up down override animation per navigation direction]] — the per-direction transition props that feed into this system
- [[Marquee component scrolls overflowing text using two synchronized looping animations]] — uses loop:true animations on x position which route through this system

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
