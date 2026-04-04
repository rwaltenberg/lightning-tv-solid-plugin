# State Order Specificity

> Config.stateOrder or per-element stateOrder controls merge priority. Later entries = higher specificity. Unmapped states have lower specificity than any mapped state.

**Source**: `solidjs-integration.md` | **Severity**: important

## Detail

When multiple states are active simultaneously, their styles are merged. The merge order is controlled by `Config.stateOrder` (global default) or a per-element `stateOrder` prop.

### Rules

1. **States listed later in `stateOrder` have higher specificity** -- they overwrite conflicting properties from earlier states.
2. **States NOT present in `stateOrder` have lower specificity than any mapped state**, regardless of their position in the `states` array.
3. **If `stateOrder` is empty** (the default `[]`), order is determined by the order states appear in the `states` array.

### Configuration

```ts
// Global default (in config):
Config.stateOrder = ['$disabled', '$active', '$focus'];
// $focus has highest specificity, $disabled has lowest

// Per-element override:
<view stateOrder={['$active', '$focus']} />
```

### Specificity Example

With `Config.stateOrder = ['$active', '$focus']`:
- `$focus` wins over `$active` for colliding props (it comes later)
- `$hover` (not in stateOrder) loses to both `$active` and `$focus`

## Code Example

```tsx
// Config.stateOrder = ['$active', '$focus']
<view
  states={['$focus', '$active', '$hover']}
  $active={{ color: 0x00ff00ff }}
  $focus={{ color: 0x0000ffff }}   // This color wins (higher specificity)
  $hover={{ alpha: 0.5 }}          // alpha applies (no collision); color ignored
/>
// Result: color = 0x0000ffff (from $focus), alpha = 0.5 (from $hover, no conflict)
```

## Gotchas

- States NOT in `stateOrder` are always lower priority than ANY state that IS in `stateOrder`, even if the unmapped state appears first in the `states` array.
- If `stateOrder` is empty, insertion order into the `states` array determines priority.
- Per-element `stateOrder` completely overrides `Config.stateOrder` for that element -- it is not merged.

## Related Notes

- [reactivity/states-system.md] -- _stateChanged() algorithm and state lifecycle
- [api/config.md] -- Config.stateOrder field definition
