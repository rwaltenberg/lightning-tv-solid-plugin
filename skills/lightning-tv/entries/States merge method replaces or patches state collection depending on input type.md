---
description: States.merge() with an array or string clears all existing states before replacing; with an object it performs selective add/remove without clearing — and neither fires onChange
type: gotcha
module: core
created: 2026-04-07
---

# States merge method replaces or patches state collection depending on input type

`States.merge()` behaves differently based on the input type, and the difference is easy to overlook:

```ts
// Array input — REPLACES all states (clears first)
states.add('$focus');
states.merge(['$disabled']); // states is now ['$disabled'] — $focus is gone

// String input — REPLACES all states (clears first)
states.merge('$disabled');   // states is now ['$disabled']

// Object input — PATCHES (additive/subtractive, no full clear)
states.add('$focus');
states.merge({ $disabled: true }); // states is now ['$focus', '$disabled']
states.merge({ $focus: false });   // states is now ['$disabled']
```

**No onChange fires from merge.** Unlike `add()` and `remove()` which fire the `onChange` callback, `merge()` modifies the array directly without triggering reactivity. This means if you use `merge()`, the node may not re-render its styles unless you separately trigger an update.

```ts
// merge() is implemented using:
this.length = 0;       // for array/string: clears without splice
this.push(...newStates);
// — no this.onChange() call
```

The object form uses `this.indexOf` and `this.splice` directly, also without calling `onChange`.

Since [[States class extends Array to manage dollar-prefixed state strings]], merge modifies the underlying Array in-place. The array reference stays the same even after a full replacement.

---

Source: [[core-states]]
Domains:
- [[core guide]]
- [[styling guide]]
