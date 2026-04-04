# States Class API Reference

> States extends Array<DollarString>. Constructor(callback, initialState?). Methods: has, is, add, remove, toggle, merge. DollarString type. has() compatibility shim checks both with and without $ prefix.

**Source**: `core-rendering-nodes.md` | **Severity**: informational

## Detail

### Type Definitions

```ts
// DollarString: template literal type requiring a $ prefix
type DollarString = `$${string}`;

// NodeStates: the accepted input types for the states prop
type NodeStates = DollarString[] | DollarString | Record<DollarString, boolean | undefined>;
```

### States Class

```ts
class States extends Array<DollarString> {
  constructor(callback: () => void, initialState?: NodeStates);
  // callback: the onChange handler, fires _stateChanged() on the ElementNode
  // initialState: optional initial state(s) to populate

  has(state: DollarString): boolean;
  // Compatibility shim: checks both `state` and `'$' + state`
  // Implementation: this.indexOf(state) >= 0 || this.indexOf('$' + state) >= 0

  is(state: DollarString): boolean;
  // Strict check (no shim): this.indexOf(state) >= 0

  add(state: DollarString): void;
  // Adds state if not already present, fires onChange

  remove(state: DollarString): void;
  // Removes state if present, fires onChange

  toggle(state: DollarString, force?: boolean): void;
  // If force is provided: add (true) or remove (false)
  // If force is omitted: adds if absent, removes if present
  // Fires onChange

  merge(newStates: NodeStates): this;
  // Replaces current states with newStates (supports all NodeStates input forms)
  // Fires onChange
  // Returns this for chaining
}
```

### has() Compatibility Shim Detail

`has()` is more lenient than `is()`. It will find `'$focus'` even if the caller passes `'focus'` (without the `$`):

```ts
// Both of these return true if '$focus' is in the array:
states.has('$focus');
states.has('focus');  // shim adds $ prefix and checks again
```

This is a backward-compatibility feature. New code should always use `$`-prefixed names.

### NodeStates Input Forms

`States.merge()` and the `states` prop accept three forms:

```ts
// Array of DollarStrings
states={['$focus', '$active']}

// Single DollarString
states="$focus"

// Record (boolean map)
states={{ $focus: isFocused(), $active: isActive() }}
// Keys with true are added, keys with false/undefined are removed
```

## Code Example

```tsx
// Access states via the getter
const node = ref;
node.states.add('$active');
node.states.toggle('$focus');
node.states.remove('$disabled');
node.states.has('$active');   // true
node.states.is('$active');    // true (strict)
node.states.has('active');    // true (shim finds '$active')

// Merge from a signal
const [activeStates, setActiveStates] = createSignal(['$focus']);
<view states={activeStates()} />
// When activeStates() changes, the states setter calls merge() internally

// Record form
<view states={{ $focus: isFocused(), $disabled: !isEnabled() }} />
```

## Gotchas

- `States` extends `Array` -- standard array methods like `forEach`, `filter`, `includes` work, but mutations (`push`, `splice`) bypass the `onChange` callback and should not be used.
- `has()` is lenient (shim), `is()` is strict. Use `has()` for compatibility, `is()` for strict checks.
- `merge()` replaces ALL current states -- it is not additive. Use `add()`/`remove()` for incremental changes.
- The `callback` passed to the constructor is `_stateChanged.bind(this)` on the ElementNode.

## Related Notes

- [reactivity/states-system.md] -- how _stateChanged() uses the States array
- [reactivity/state-order-specificity.md] -- how stateOrder affects _stateChanged merge order
- [api/element-node.md] -- states getter/setter on ElementNode
