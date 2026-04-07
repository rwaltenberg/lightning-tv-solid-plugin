# Shader Property Transitions Are Broken

> border, shadow, rounded, borderTop/Right/Bottom/Left use a separate shader animation path that is broken for `transition={true}` and `transition={{ border: true }}`. linearGradient and radialGradient have NO transition support at all.

**Source**: `elementNode.ts` shaderAccessor (lines 1518-1577) | **Severity**: critical

## Detail

Shader properties (border, shadow, rounded, borderRadius, borderTop, borderRight, borderBottom, borderLeft) use `shaderAccessor()`, which has a different transition path from regular animatable properties.

### Why Shader Transitions Fail

Regular properties use `_sendToLightningAnimatable`, which falls through to defaults:

```ts
// Regular properties -- WORKS with transition={true}
this.lng.animateProp(name, value, animationSettings || this.animationSettings || {});
```

Shader properties have a broken guard:

```ts
// Shader properties -- FAILS with transition={true}
if (animationSettings) {  // undefined is falsy → animation never fires
  this.animate({ shaderProps: target }, animationSettings).start();
}
```

When `transition={true}` or `transition={{ border: true }}`, `animationSettings` is set to `undefined`, and the `if (animationSettings)` check prevents the animation from ever starting.

### Additional Prerequisite: Shader Must Already Exist

The transition code path is inside `if (this.lng.shader?.props)` -- meaning the shader object must already be initialized with props. The first time you set a border/shadow/rounded, the shader doesn't exist yet, so transitions are skipped entirely regardless of settings.

### linearGradient and radialGradient

These use `createRawShaderAccessor()` which has **zero** transition logic. They cannot be transitioned at all -- they are immediate shader replacements.

## What To Do Instead

For dynamic shader effects, use states with immediate switches (no transition):

```tsx
const cardStyle: NodeStyles = {
  border: { width: 0, color: 0xffffffff },
  $focus: { border: { width: 4, color: 0x00ff00ff } },
};

<view style={cardStyle} />
```

Or use `animate()` directly in a lifecycle callback:

```tsx
<view
  border={{ width: 0, color: 0xffffffff }}
  onCreate={(el) => {
    el.animate(
      { shaderProps: { 'border-width': 4 } },
      { duration: 300, easing: 'ease-in-out' }
    ).start();
  }}
/>
```

Or animate a wrapper's alpha/scale to simulate a transition instead of animating the border itself.

## Gotchas

- Do NOT use `transition={{ border: { duration: 300 } }}` expecting it to work reliably -- even with explicit settings, the shader must already be initialized.
- `rounded`/`borderRadius` maps to transition key `borderRadius` internally, but still uses the broken shader path.
- The `BorderBox` primitive from `@lightningtv/solid/primitives` transitions `alpha` (a regular animatable prop), NOT the border itself -- this is the recommended pattern.

- **`border-gap` doesn't work with combined border+shadow.** The `RoundedWithBorderAndShadow` shader doesn't support the `border-gap` property (`shaders.ts:67` has an explicit TODO). The TypeScript type claims it does via a type cast, so there's no compile-time warning -- it just silently has no effect.

## Related Notes

- [reactivity/reactive-transitions.md] -- regular property transitions (which DO work)
- [constraints/animate-before-render.md] -- animate() requires rendered === true
- [api/element-node.md] -- shader/effects accessor list
