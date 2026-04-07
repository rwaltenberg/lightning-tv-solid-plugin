---
description: shaderAccessor only animates when animationSettings resolves to a truthy value; if transition entry is true and no fallback exists the animation is skipped without error
type: gotcha
module: core
created: 2026-04-07
---

# shader transitions fail silently when animationSettings is undefined

The `shaderAccessor` function — which backs `border`, `shadow`, `rounded`, `borderRadius`, and `borderTop/Bottom/Left/Right` — has a guarded animation path that silently skips animation when `animationSettings` resolves to `undefined`.

## Source

```typescript
// elementNode.ts — shaderAccessor setter (simplified)
set(this: ElementNode, value: T) {
  let animationSettings: AnimationSettings | undefined;

  if (this.lng.shader?.props) {
    const transitionKey = key === 'rounded' ? 'borderRadius' : key;
    if (
      this.transition &&
      (this.transition === true || this.transition[transitionKey])
    ) {
      target = {};
      animationSettings =
        this.transition === true || this.transition[transitionKey] === true
          ? undefined          // <-- falls through as undefined
          : (this.transition[transitionKey] as undefined | AnimationSettings);
    }
  }

  // ...apply shader props...

  if (animationSettings) {          // <-- undefined fails this check
    this.animate({ shaderProps: target }, animationSettings).start();
  }
}
```

When `this.transition === true` (animate everything with default settings) or `this.transition[transitionKey] === true` (animate this specific property with defaults), `animationSettings` is set to `undefined`. The final `if (animationSettings)` then skips the `animate()` call entirely.

## Why this happens

The intent is to let `animationSettings` fall back to the node's default `animationSettings` via the `animate()` method itself. But `shaderAccessor` never passes `undefined` to `animate()` — it only calls `animate()` if `animationSettings` is truthy. This means:

- `this.transition = true` → shader props update **instantly**, never animated
- `this.transition = { border: true }` → border updates **instantly**
- `this.transition = { border: { duration: 300 } }` → border **animates correctly**

Contrast with regular animatable props via `_sendToLightningAnimatable`, which does pass through to `animateProp` with `animationSettings || this.animationSettings || {}`:

```typescript
// Regular props — properly falls back to node animationSettings
return (this.lng as any).animateProp(
  name,
  value,
  animationSettings || this.animationSettings || {},
);
```

## Workaround

Always provide explicit `AnimationSettings` objects for shader properties rather than using `true`:

```tsx
// Broken — border never animates even though transition is set
<view transition={true} border={{ width: 2, color: 0xff0000ff }} />

// Broken — same problem with property-level true
<view transition={{ border: true }} border={{ width: 2, color: 0xff0000ff }} />

// Fixed — explicit duration causes animationSettings to be truthy
<view
  transition={{ border: { duration: 300, easing: 'ease-in-out' } }}
  border={{ width: 2, color: 0xff0000ff }}
/>
```

Also note that animation only triggers if `this.lng.shader?.props` is already set — first render always applies immediately regardless of transition setting.

---

Related Entries:
- [[animatable number properties route through transition system before reaching the renderer]] — the parallel path for number props that correctly falls back to animationSettings
- [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]] — the animate() method called when animationSettings is truthy
- [[effects setter builds shader type by composing feature flags and converts before render]] — how the shader itself is constructed before shaderAccessor animates it

Domains:
- [[styling guide]]
- [[core guide]]
