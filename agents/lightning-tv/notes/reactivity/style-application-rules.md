# Style Application Rules

> The style setter runs once and copies keys only if not already defined on the element. JSX props take priority. Config.lockStyles=true prevents re-set.

**Source**: `solidjs-integration.md` | **Severity**: critical

## Detail

Styles are applied at creation time. The `style` setter iterates all keys from the style object and sets them on the `ElementNode` **only if the element does not already have a value for that key**. This means:

- **JSX props always take priority over style props** -- properties set directly in JSX are applied first (before `style` is processed), so style keys that conflict are silently skipped.
- **`Config.lockStyles = true`** (the default) means once `_style` is set on a node, subsequent style assignments are silently dropped. The code explicitly warns in dev mode: `'Style already set'`.
- Style is designed to be set **exactly once** as a static base. It is not reactive.

### Priority Order (highest to lowest)

1. Individual JSX props (e.g., `x={100}`)
2. Style object keys (e.g., `style={{ x: 0 }}`)

### Dev Mode Warning

When `Config.lockStyles` is true and `style` is assigned a second time, a dev mode warning fires: `'Style already set'`. In production this is a silent no-op.

## Code Example

```tsx
// JSX prop takes priority over style
<view x={100} style={{ x: 0, color: 0xff0000ff }} />
// Result: x = 100 (from JSX prop), color = 0xff0000ff (from style)

// WRONG: style with a reactive signal is wasteful and will not update
<view style={dynamicSignal()} />

// CORRECT: use individual reactive props for dynamic values
<view color={dynamicColor()} x={dynamicX()} />

// CORRECT: use states for predefined visual variants
<view
  style={baseStyle}
  states={activeStates()}
  $focus={{ color: 0x0000ffff }}
/>
```

## Gotchas

- `Config.lockStyles` defaults to `true` -- style is locked after first set.
- Wrapping `style` in a reactive signal is a no-op after the first render; subsequent signal updates are silently ignored.
- There is no CSS cascade -- the style object is a flat key-value map.
- Style keys with `$`-prefix are state-style overrides, not regular style keys.

## Related Notes

- [constraints/style-locked-once.md] -- full constraint note on lockStyles
- [reactivity/states-system.md] -- the correct way to do dynamic styling via states
- [reactivity/flatten-styles-first-wins.md] -- how flattenStyles merges style arrays
