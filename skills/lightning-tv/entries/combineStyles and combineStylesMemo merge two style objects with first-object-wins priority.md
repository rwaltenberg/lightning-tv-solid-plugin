---
description: combineStyles does a plain spread merge; combineStylesMemo returns a reactive Accessor<T> using createMemo only when both styles are defined
type: api
module: utils
created: 2026-04-07
---

# combineStyles and combineStylesMemo merge two style objects with first-object-wins priority

Two utility functions for merging Lightning style objects, both using `style1` as the higher-priority input:

```ts
export function combineStyles<T extends Styles>(style1: T | undefined, style2: T | undefined): T {
  if (!style1) return style2!;
  if (!style2) return style1;
  return { ...style2, ...style1 }; // style1 wins
}
```

`combineStyles` short-circuits when either argument is undefined, avoiding unnecessary object creation. When both are defined, style1's properties override style2's.

```ts
export function combineStylesMemo<T extends Styles>(style1: T | undefined, style2: T | undefined): Accessor<T> {
  if (!style1) return () => style2!;
  if (!style2) return () => style1;
  return createMemo(() => ({ ...style2, ...style1 }));
}
```

`combineStylesMemo` returns an `Accessor<T>` for use in reactive contexts. When one style is undefined, it returns a plain arrow function (no reactive overhead). A `createMemo` is only created when both styles are defined, making the merged result reactive to changes in either.

Contrast with [[flattenStyles merges style arrays with first-value-wins priority]] which handles arrays of styles. These functions handle the two-style case and are typically used in component implementations where a base style and an override style need to be combined.

Since [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]], Row uses `combineStyles(props.style, RowStyles)` so user-provided styles take precedence over the built-in defaults — the same pattern applies to Column.

---

Source: [[utils]]
Domains:
- [[core guide]]
- [[styling guide]]
