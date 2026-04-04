# States System

> $-prefixed keys define state styles. _stateChanged() undoes previous states, sorts by stateOrder, and merges via Object.assign. forwardStates propagates to children.

**Source**: `solidjs-integration.md`, `core-rendering-nodes.md` | **Severity**: critical

## Detail

Each `ElementNode` has an optional `_states: States` array. `States` extends `Array<DollarString>`.

State names must begin with `$`. The type system enforces `` `$${string}` `` (the `DollarString` type).

### State Style Definition

State styles are defined as `$`-prefixed props directly on the element (typically via the `style` object):

```tsx
<view
  color={0x000000ff}
  $focus={{ color: 0xff0000ff, scale: 1.1 }}
  $disabled={{ alpha: 0.5 }}
/>
```

### `_stateChanged()` Algorithm

When states change (via `add`, `remove`, `toggle`, `merge`), the `onChange` callback fires `_stateChanged()`, which:

1. **Forwards states to children** if `forwardStates === true`.
2. **Undoes previously applied state styles** -- restores base style values.
3. **Sorts active states by `stateOrder`** (from `Config.stateOrder` or per-element `stateOrder`). States later in the order have higher specificity.
4. **Merges state styles** left-to-right so higher-specificity states overwrite lower ones.
5. **Applies** the merged style object via `Object.assign(this, newStyles)`.

### `forwardStates`

Setting `forwardStates={true}` on a parent causes state changes to propagate automatically to all children.

### States Class

```ts
class States extends Array<DollarString> {
  constructor(callback: () => void, initialState?: NodeStates);
  has(state: DollarString): boolean;
  is(state: DollarString): boolean;
  add(state: DollarString): void;
  remove(state: DollarString): void;
  toggle(state: DollarString, force?: boolean): void;
  merge(newStates: NodeStates): this;
}
```

### `NodeStates` Type

```ts
type NodeStates = DollarString[] | DollarString | Record<DollarString, boolean | undefined>;
```

## Code Example

```tsx
// State styles defined in the style object
const MyComponent = () => {
  return (
    <view
      style={{
        color: 0x000000ff,
        $focus: { color: 0xff0000ff, scale: 1.1 },
        $disabled: { alpha: 0.5 },
      }}
      states={{ $focus: isFocused(), $disabled: isDisabled() }}
    />
  );
};

// Programmatic state manipulation
const node = ref;
node.states.add('$active');
node.states.toggle('$focus');
node.states.remove('$disabled');

// Forward states to children
<view forwardStates states={['$focus']}>
  <view $focus={{ color: 0x0000ffff }} />  // receives $focus automatically
</view>
```

## Gotchas

- State names MUST begin with `$`. `states={['focus']}` (no `$`) will not match `$focus` style key.
- `has()` has a compatibility shim: `this.indexOf(state) >= 0 || this.indexOf('$' + state) >= 0` -- checks both with and without `$` prefix.
- `_stateChanged()` calls `Object.assign(this, newStyles)` which means state styles can overwrite any property on the node.
- State merge order follows `stateOrder` -- see state-order-specificity note for details.

## Related Notes

- [reactivity/state-order-specificity.md] -- how stateOrder controls merge priority
- [api/states-class.md] -- full States class API reference
- [reactivity/style-application-rules.md] -- styles are static base; states provide dynamic overrides
