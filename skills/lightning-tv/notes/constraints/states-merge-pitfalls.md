# States merge() Pitfalls

> states.merge() does NOT trigger onChange / _stateChanged -- visual styles won't update. Array and string forms clear ALL existing states before adding new ones.

**Source**: `states.ts` (lines 59-82) | **Severity**: critical

## Detail

### merge() Does Not Trigger Visual Updates

Unlike `add()` and `remove()`, `merge()` never calls `this.onChange()`. This means calling `node.states.merge(...)` modifies the internal state array but **does NOT trigger `_stateChanged()`**, so styles associated with the new states won't be applied.

```ts
// add() calls onChange -- styles update
node.states.add('$focus');  // WORKS: triggers _stateChanged()

// merge() does NOT call onChange -- styles DO NOT update
node.states.merge(['$focus']);  // BROKEN: state array changes, visuals don't
```

### Array/String Form Clears All States

The array and string forms of `merge()` call `this.length = 0` before pushing new values. This **wipes every existing state**.

```ts
// Object form: additive -- only adds/removes named states
node.states.merge({ $active: true, $disabled: false });
// Result: $active added, $disabled removed, everything else untouched

// Array form: destructive -- clears everything first
node.states.merge(['$active']);
// Result: ONLY $active remains. $focus, $hover, etc. are ALL gone

// String form: also destructive
node.states.merge('$active');
// Result: ONLY $active remains
```

### Safe Patterns

Use `add()` and `remove()` for individual state changes (they call `onChange`):

```tsx
node.states.add('$active');     // adds state, triggers visual update
node.states.remove('$focus');   // removes state, triggers visual update
node.states.toggle('$active');  // toggles state, triggers visual update
```

If you need to replace all states at once, assign through the setter instead:

```tsx
node.states = ['$active', '$focus'];  // Creates new States object, triggers update
```

## Gotchas

- `merge()` is safe for building up state during initialization (before render), but dangerous after render because visuals won't update.
- The object form `{ $state: true/false }` doesn't clear, but still doesn't call `onChange`.
- `has()` checks both `$state` and `state` (with/without prefix) for backwards compatibility, but `merge`/`add`/`remove` do not normalize -- always use the `$` prefix.

## Related Notes

- [reactivity/states-system.md] -- $-prefixed keys, _stateChanged, Object.assign merge
- [reactivity/state-order-specificity.md] -- merge order affects which state wins
