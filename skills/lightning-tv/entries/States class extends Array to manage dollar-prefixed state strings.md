---
description: States extends the native Array class to hold active DollarString state names on an ElementNode, with add/remove/toggle/merge methods that fire an onChange callback
type: architecture
module: core
created: 2026-04-07
---

# States class extends Array to manage dollar-prefixed state strings

`States` is the class behind `ElementNode.states`. It extends `Array<DollarString>` so all native array methods (forEach, includes, spread, etc.) work directly on it.

```ts
// Typical usage through ElementNode
node.states.add('$focus');
node.states.remove('$disabled');
node.states.toggle('$hover');

// It's a real array
console.log([...node.states]); // ['$focus']
node.states.forEach(s => console.log(s));
```

**Initialization formats:**

```ts
// Any of these work as initialState:
new States(cb, ['$focus', '$disabled'])         // array
new States(cb, '$focus')                         // single string
new States(cb, { $focus: true, $disabled: false }) // object map
```

**Methods:**

| Method | Behavior |
|---|---|
| `has(state)` | Permissive — matches `state` OR `$state` |
| `is(state)` | Strict — exact indexOf match only |
| `add(state)` | Deduplicates, then pushes + fires onChange |
| `remove(state)` | Splices if found, fires onChange |
| `toggle(state, force?)` | `force=true` adds, `force=false` removes, no force flips |
| `merge(newStates)` | Batch update — see [[States merge method replaces or patches state collection depending on input type]] |

Since [[States has method tolerates dollar prefix in both directions]], `states.has('focus')` matches an element with state `'$focus'` — useful for legacy compatibility.

Since [[Config.focusStateKey controls which state string marks focused elements]], the focus state key is configurable and `States.has()` is used internally to check it.

---

Source: [[core-states]]
Domains:
- [[core guide]]
- [[styling guide]]
