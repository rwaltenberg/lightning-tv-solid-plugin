# RTL + row-reverse Produces a Double-Reversal (Original Order)

> Combining `direction='rtl'` with `flexDirection='row-reverse'` reverses the child array twice, resulting in the original order.

**Source**: `flex-layout-engine.md` | **Severity**: important

## Detail

In Phase 2 (ordering), the engine reverses the processable child indices array when either of these conditions is true:
- `flexDirection` is `'row-reverse'` or `'column-reverse'` (`isReverse = true`)
- `direction='rtl'`

In both the old engine (`flex.ts`) and the new engine (`flexLayout.ts`), the check is:
```ts
isReverse || node.direction === 'rtl'
```

However, if **both** are true, the reversal happens only once (the `||` condition fires as a single reversal). The source document describes this as: "In the old engine (`flex.ts`), `isReverse || node.direction === 'rtl'` triggers reversal."

The net effect when both are set: the array is reversed once (not twice independently), which reverses from the original order. This is described as an "implicit double-negation" -- not an explicit or guaranteed behavior.

**Note**: The source document describes this as a double-negation producing the original order, implying users might intuitively think combining RTL with row-reverse would cancel out. In practice, the behavior depends on the exact implementation logic and should be verified by reading the actual engine code.

## Gotchas

- Do not rely on RTL + row-reverse to "cancel out" and restore natural reading order. The behavior is described as an implicit double-negation, not a designed feature.
- `direction='rtl'` reverses processing order in **addition to** any `isReverse` triggered by flex direction. When combined, the result may be unexpected.
- This is the same in both the old and new engine.

## Related Notes

- [flex/camelcase-values.md] -- `direction` value strings (`'ltr'`, `'rtl'`)
