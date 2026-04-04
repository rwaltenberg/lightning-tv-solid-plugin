# Padding Array Support is New Engine Only

> The old engine casts `padding` to a single number. Only the new engine supports `[T,R,B,L]` array form and per-side overrides.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

**Old engine (`flex.ts`)**: casts `padding` to a plain number:
```ts
const nodePadding = (node.padding as number) || 0;
```
Array values will be coerced and produce incorrect layout. Only a single numeric `padding` value is safe with the old engine.

**New engine (`flexLayout.ts`, active when `VITE_USE_NEW_FLEX` is set)**: supports the full array syntax and per-side overrides:

`padding` accepts:
- `number` -- all sides
- `[number, number]` -- interpreted as `[T, B]`
- `[number, number, number]` -- interpreted as `[T, H, B]`
- `[number, number, number, number]` -- interpreted as `[T, R, B, L]`

Per-side overrides (new engine only):
- `paddingTop`
- `paddingRight`
- `paddingBottom`
- `paddingLeft`

Per-side properties override the corresponding value from the `padding` array when both are set.

The new engine also adds `margin` array shorthand for children: `[T, R, B, L]`. The old engine only supports individual `marginLeft`, `marginRight`, `marginTop`, `marginBottom` properties.

## Gotchas

- Using an array padding value with the old engine will coerce to a non-zero number unexpectedly (JavaScript array-to-number coercion is `NaN` for multi-element arrays, which becomes `0` via `|| 0`). This silently zeroes out padding.
- There is no validation or warning when array padding is used with the old engine.

## Related Notes

- [flex/flexshrink-flexbasis-new-engine.md] -- other new-engine-only properties
