---
description: stateOrder is a DollarString[] on Config that sets explicit priority for state-based style variants when multiple states are active simultaneously
type: api
module: core
created: 2026-04-07
---

# Config.stateOrder controls style resolution priority when multiple states are active

When an element has multiple active states (e.g., `$focus` and `$disabled`), `Config.stateOrder` determines which state's style variant takes priority. It is an ordered array of `DollarString` values.

```ts
// Default — no explicit ordering
Config.stateOrder = [];

// Explicit priority: $disabled wins over $focus wins over $hover
Config.stateOrder = ['$disabled', '$focus', '$hover'];
```

With an empty `stateOrder`, the priority behavior depends on the style resolution implementation (likely first-match or last-match). Setting an explicit order makes multi-state styling deterministic.

This is especially important for TV apps where elements can be simultaneously focused and in a loading/disabled state and the correct visual state must be predictable.

Since [[Config singleton holds all runtime Lightning configuration]], `stateOrder` is set globally and affects all elements.

Since [[States class extends Array to manage dollar-prefixed state strings]], the active states on each element are a `States` array — `stateOrder` is compared against the contents of this array during style resolution.

---

Source: [[core-config]]
Domains:
- [[core guide]]
- [[styling guide]]
