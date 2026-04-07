---
description: The focusStateKey field (default '$focus') determines the DollarString added to element states when focused, used by isFocused() and style variant resolution
type: api
module: core
created: 2026-04-07
---

# Config.focusStateKey controls which state string marks focused elements

`Config.focusStateKey` is a `DollarString` (string starting with `$`) that defaults to `'$focus'`. When an element receives focus, this string is added to its `States` array. Removing focus removes the string.

The `isFocused()` utility in `utils.ts` checks for exactly this key:

```ts
export function isFocused(el: ElementNode | ElementText): boolean {
  return el?.states?.has(Config.focusStateKey);
}
export const hasFocus = isFocused; // alias
```

Style variants use this key directly:

```ts
// In your component styles
const buttonStyle = {
  color: 0xffffffff,
  $focus: {
    color: 0x00ff00ff,
  },
};
```

If you change `Config.focusStateKey` to something like `'$active'`, all focus-triggered style variants must be renamed to `$active` across your app, and `isFocused()` will check for `$active`.

Since [[States has method tolerates dollar prefix in both directions]], `states.has('focus')` would also match `$focus` — but the key used in style variants must match the Config value exactly.

---

Source: [[core-config]], [[core-utils]]
Domains:
- [[core guide]]
- [[styling guide]]
