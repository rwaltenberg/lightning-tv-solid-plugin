---
description: utils.ts exports both isFunc (instanceof Function) and isFunction (typeof function) as separate type guards — they are not interchangeable and exist for different use cases
type: gotcha
module: core
created: 2026-04-07
---

# isFunc and isFunction are intentionally different type guards

`utils.ts` exports two function type guards with subtly different semantics:

```ts
// instanceof check — narrower, catches callable objects
export const isFunc = (obj: unknown): obj is CallableFunction =>
  obj instanceof Function;

// typeof check — broader, standard JS idiom
export const isFunction = (obj: unknown): obj is Function =>
  typeof obj === 'function';
```

**Differences:**
- `isFunc` returns `CallableFunction` (can be safely called with `()`) vs `isFunction` returns `Function` (not guaranteed callable without the right signature)
- `instanceof Function` fails across iframe boundaries (different Function constructors) — `typeof` does not
- For most Lightning TV use cases (same-origin, single window), they behave identically
- `isFunc` implies the function is directly invocable; `isFunction` is a looser check

**When each is used internally:**
- `isFunc` — used for checking if style properties or callbacks are functions before invoking them
- `isFunction` — used in broader type narrowing contexts

In practice, use `isFunc` when you need to call the result and want the TypeScript type to reflect that. Use `isFunction` for general function detection.

Since [[Config singleton holds all runtime Lightning configuration]] has `convertToShader` and `setActiveElement` as function fields, these guards are used when these callbacks are invoked.

---

Source: [[core-utils]]
Domains:
- [[core guide]]
