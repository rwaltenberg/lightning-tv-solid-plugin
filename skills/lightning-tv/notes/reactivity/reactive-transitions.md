# Reactive Transitions

> The transition prop enables animated property changes. Record<string, AnimationSettings | true | false> | true | false. Default: duration 250, easing 'ease-in-out'.

**Source**: `solidjs-integration.md` | **Severity**: important

## Detail

The `transition` prop configures which properties animate when changed reactively. When a signal drives an animatable property and `transition` is configured for that property, `_sendToLightningAnimatable` calls `this.lng.animateProp(name, value, settings)` instead of setting directly.

### Type

```ts
transition: Record<string, AnimationSettings | true | false> | true | false
```

### Default Animation Settings

```ts
animationSettings: {
  duration: 250,
  easing: 'ease-in-out'
}
```

### Behavior

- `transition={true}` -- enables transitions on ALL animatable properties using default animation settings.
- `transition={false}` -- disables all transitions.
- `transition={{ x: { duration: 300 }, y: false }}` -- per-property configuration. `true` uses default settings. `false` disables for that property.

### Alias Resolution

`_sendToLightningAnimatable` resolves aliases before checking `transition`. Width/height aliases (`w`/`width`, `h`/`height`) are resolved to their canonical names.

### animationSettings Prop

The `animationSettings` prop on an element provides default settings that apply when a transition key is `true`:

```ts
animationSettings?: Partial<AnimationSettings>
```

## Code Example

```tsx
const [xPos, setXPos] = createSignal(0);

// Per-property transition settings
<view
  x={xPos()}
  transition={{ x: { duration: 300, easing: 'ease-out' } }}
/>

// Enable all animatable props with defaults
<view
  x={xPos()}
  color={myColor()}
  transition={true}
/>

// Mix: some enabled, some not
<view
  x={xPos()}
  y={yPos()}
  transition={{ x: { duration: 500 }, y: false }}
/>

// Custom default settings
<view
  x={xPos()}
  animationSettings={{ duration: 400, easing: 'ease-in' }}
  transition={{ x: true }}
/>
```

## Gotchas

- Transitions only fire AFTER the node is rendered (`rendered === true`). Pre-render property sets are always direct writes even if `transition` is configured.
- Only **animatable** properties (numeric props like x, y, w, h, color, alpha, scale, rotation, etc.) support transitions. Non-animating props (text, clipping, etc.) ignore `transition`.
- `transition={true}` enables all animatable props -- be careful about unintended animations on initial mount.

## Related Notes

- [reactivity/signal-to-gpu-pipeline.md] -- the full signal -> animateProp path
- [core/element-node-proxy.md] -- which properties are animatable
- [constraints/animate-before-render.md] -- transitions only fire after render
