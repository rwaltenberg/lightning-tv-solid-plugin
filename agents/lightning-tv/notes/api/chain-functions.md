# chainFunctions / chainRefs

> Composes multiple event handler functions into a chain that stops on the first truthy return.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`chainFunctions` chains functions together. If any function in the chain returns `true`, the chain stops (the event is consumed). This is the primary extension mechanism for composing navigation handlers on `onLeft`, `onRight`, `onUp`, `onDown`, and `onSelectedChanged` props. Accepts `undefined`, `null`, or `false` values in the chain (they are skipped).

`chainRefs` is for SolidJS `ref` prop forwarding -- it composes multiple ref callbacks so that a single `ref` prop can assign the element to multiple variables.

**Export**:
```ts
export function chainFunctions<T extends AnyFunction>(...fns: (T | undefined | null | false)[]): T | undefined
export const chainRefs: <T>(...refs: (Ref<T> | undefined)[]) => (el: T) => void
```

## Props / API

```ts
chainFunctions<T extends AnyFunction>(...fns: (T | undefined | null | false)[]): T | undefined
chainRefs<T>(...refs: (Ref<T> | undefined)[]): (el: T) => void
```

- `chainFunctions`: variadic -- accepts any number of functions (or falsy values). Returns a composed function or `undefined` if all inputs are falsy.
- `chainRefs`: variadic -- accepts any number of SolidJS ref callbacks. Returns a single ref callback that calls all of them.

## Gotchas

- If any handler returns `true`, the chain STOPS -- subsequent handlers do NOT run. This is how custom handlers can "consume" a navigation event before the default Row/Column behavior.
- Passing `undefined`, `null`, or `false` is safe -- they are skipped.
- `onSelectedChanged` is NOT part of the key handler chain and does NOT support `return true` to consume. Only `onLeft`/`onRight`/`onUp`/`onDown` key handlers support this.
- User-provided handlers run FIRST in Row/Column (via chainFunctions), before the default navigation handler.

## Related Notes

- [api/row.md] -- uses chainFunctions for onLeft/onRight
- [api/column.md] -- uses chainFunctions for onUp/onDown
- [api/use-hold.md] -- produces handlers suitable for chainFunctions
