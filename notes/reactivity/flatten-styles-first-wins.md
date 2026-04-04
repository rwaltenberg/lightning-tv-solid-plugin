# flattenStyles First-Wins Semantics

> flattenStyles uses first-wins: earlier array entries have higher priority. Opposite of typical spread behavior.

**Source**: `core-rendering-nodes.md` | **Severity**: important

## Detail

`flattenStyles` (from `utils.ts`) merges arrays of style objects with **first-wins** semantics: if `result[key]` is already defined (not `undefined`), it is NOT overwritten by a later entry.

This is the opposite of typical spread behavior (`{ ...a, ...b }` where `b` wins).

### Signature

```ts
export function flattenStyles(
  obj: Styles | undefined | (Styles | undefined)[],
  result?: Styles
): Styles;
```

### Priority Rule

Earlier entries in the array have **higher priority**. This is intentional -- it means:
- Inline/specific styles at the front of the array override base/shared styles at the back.
- When used internally by the framework, more specific props override less specific ones.

## Code Example

```ts
import { flattenStyles } from '@lightningtv/solid';

const highPriority = { color: 0xff0000ff, x: 100 };
const lowPriority  = { color: 0x00ff00ff, y: 200 };

const result = flattenStyles([highPriority, lowPriority]);
// result.color = 0xff0000ff  (highPriority wins -- first entry)
// result.x     = 100         (only in highPriority)
// result.y     = 200         (only in lowPriority)

// NOT like spread:
// { ...highPriority, ...lowPriority }.color === 0x00ff00ff  (lowPriority wins in spread)
```

## Gotchas

- If you pass `[baseStyle, overrideStyle]` expecting `overrideStyle` to win, it will NOT. The order must be `[overrideStyle, baseStyle]` for first-wins semantics.
- Keys with `undefined` values are NOT considered "defined" -- they can be overwritten.

## Related Notes

- [reactivity/style-application-rules.md] -- how style is applied (also first-wins: JSX props set before style runs)
- [api/element-node.md] -- flattenStyles usage in the ElementNode codebase
