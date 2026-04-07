---
description: flex.ts uses a single || condition for isReverse and RTL, so combining flexDirection=row-reverse with direction=rtl reverses the child array only once, not twice
type: gotcha
module: core
created: 2026-04-07
---

# RTL direction combined with row-reverse produces only a single reversal not a double

The flex layout engine applies child order reversal through a single `if` block that combines `isReverse` (from `flexDirection: row-reverse` or `column-reverse`) and `node.direction === 'rtl'` using `||`. When both are active simultaneously, the reversal only happens once — resulting in the same visual order as applying either one alone, not a double reversal back to the original order.

## Source

```typescript
// flex.ts
const isReverse =
  direction === 'row-reverse' || direction === 'column-reverse';

// ... children collected into processableChildrenIndices ...

if (isReverse || node.direction === 'rtl') {
  processableChildrenIndices.reverse();  // called exactly once
}
```

## What this means in practice

| flexDirection | direction | Expected (CSS) | Actual |
|--------------|-----------|----------------|--------|
| `row` | `ltr` | [A, B, C] | [A, B, C] |
| `row-reverse` | `ltr` | [C, B, A] | [C, B, A] |
| `row` | `rtl` | [C, B, A] | [C, B, A] |
| `row-reverse` | `rtl` | [A, B, C] | [C, B, A] ← wrong |

The last row is the gotcha. Standard CSS flexbox with `flex-direction: row-reverse` and `direction: rtl` would cancel each other and show `[A, B, C]` (left-to-right). The Lightning TV flex engine shows `[C, B, A]` (right-to-left) because the `||` guards prevent a second reversal call.

## Root cause

Both RTL and row-reverse translate into the same index-reversal operation on `processableChildrenIndices`. The engine treats them as aliases. Combining them does not compound — it still calls `.reverse()` once.

## Workaround

Do not combine `flexDirection: 'row-reverse'` with `direction: 'rtl'` expecting a double-reversal. If you need the original LTR order in an RTL context, use `flexDirection: 'row'` (the RTL reversal alone will give you right-to-left). If you want the row-reverse order in RTL, use `flexDirection: 'row-reverse'` alone and avoid the `direction: 'rtl'` prop.

---

Related Entries:
- [[flexDirection controls main axis orientation and supports row-reverse and column-reverse]] — the description of how reverse variants work in the engine; mentions RTL using the same code path
- [[flex layout engine positions children using justifyContent along the main axis]] — the broader flex layout algorithm context
- [[flexOrder reorders children visually without changing DOM order]] — another child-ordering mechanism that runs before the reversal step

Domains:
- [[styling guide]]
- [[core guide]]
