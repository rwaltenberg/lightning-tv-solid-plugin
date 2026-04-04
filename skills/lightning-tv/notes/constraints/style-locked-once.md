# Style Locked Once

> Config.lockStyles defaults to true. The style setter is a no-op after first set. Dev mode warns 'Style already set'. Never use reactive signals for the style prop.

**Source**: `solidjs-integration.md`, `core-rendering-nodes.md` | **Severity**: critical

## Detail

`Config.lockStyles` defaults to `true`. Once `_style` is set on an `ElementNode`, subsequent style assignments are:
- **Silently dropped** in production.
- **Warned** in dev mode with: `'Style already set'`.

This is deliberate -- styles are intended to be set once as a static base. The framework is not designed for dynamic style objects.

### Why This Matters

Using a reactive signal as the `style` prop will silently fail after the first render:

```tsx
// WRONG: Only the initial value of dynamicStyle() is applied.
// Subsequent signal updates are silently ignored.
<view style={dynamicStyle()} />
```

### Correct Alternatives

- Use **individual reactive props** for properties that change dynamically.
- Use the **states system** for predefined visual variants driven by state.

## Code Example

```tsx
// WRONG -- second assignment is silently dropped
<view style={styleA} />
// Later in code: node.style = styleB; // NO EFFECT, warns in dev mode

// WRONG -- reactive signal on style prop
<view style={dynamicSignal()} />

// CORRECT -- reactive individual props
<view color={dynamicColor()} x={dynamicX()} />

// CORRECT -- static base style + states for variants
<view
  style={{ color: 0x000000ff, $focus: { color: 0xff0000ff } }}
  states={activeStates()}
/>
```

## Gotchas

- `Config.lockStyles` can be set to `false` to disable the lock, but this is not recommended.
- Even with `lockStyles: false`, the "first-wins" semantics of the style setter still apply (keys already set on the element from JSX props are not overwritten by style).
- The warning in dev mode helps catch accidental re-assignments.

## Related Notes

- [reactivity/style-application-rules.md] -- full style application mechanics (first-wins, JSX priority)
- [reactivity/states-system.md] -- the correct pattern for dynamic styling
- [api/config.md] -- Config.lockStyles field
