---
description: States.has() checks for both 'state' and '$state' variants, while States.is() requires an exact match — has() exists for backward compatibility with unprefixed state names
type: gotcha
module: core
created: 2026-04-07
---

# States has method tolerates dollar prefix in both directions

`States` exposes two presence-check methods with different strictness:

```ts
states.add('$focus');

states.has('$focus'); // true — direct match
states.has('focus');  // ALSO true — tries '$focus' as fallback
states.is('$focus');  // true — exact match
states.is('focus');   // false — strict indexOf, no prefix tolerance
```

The `has()` implementation:
```ts
has(state: DollarString) {
  // temporary check for $ prefix
  return this.indexOf(state) >= 0 || this.indexOf(`$${state}`) >= 0;
}
```

The source comment says "temporary check for $ prefix" — this is a backward-compatibility shim, not a permanent design feature. Code that relies on `has('focus')` matching `'$focus'` may break in a future version when this tolerance is removed.

**Best practice:** Always use the full `$`-prefixed state name in both `add()` calls and `has()` checks. Don't rely on the prefix tolerance.

Since [[States class extends Array to manage dollar-prefixed state strings]], `is()` uses the base `indexOf` directly and has no such tolerance.

Since [[Config.focusStateKey controls which state string marks focused elements]], the framework internally uses `states.has(Config.focusStateKey)` with the full key.

---

Source: [[core-states]]
Domains:
- [[core guide]]
- [[styling guide]]
