---
description: LazyRow and LazyColumn use Solid's `<Index>` for rendering so items are keyed by array position; appending new items does not cause existing items to remount
type: gotcha
module: primitives
created: 2026-04-07
---

The choice between `<For>` and `<Index>` for list rendering has significant performance implications in Lazy components:

```ts
// Lazy uses:
<s.Index each={items()} children={props.children} />

// Virtual uses:
<List each={slice().slice}>{props.children}</List>
```

## Why Index

`<Index>` keys items by position. When a new item is appended to the array (offset grows by 1), only the new item at the last position is created. Existing items at positions 0..N-1 receive a new `item` accessor value if the array contents change at their position, but the DOM node is not destroyed and recreated.

`<For>` keys by identity. If the items array is replaced (even with the same values), `<For>` destroys and recreates all items. For Lazy's use case — where items are appended and never reordered — `<Index>` is strictly better.

## Implication for children prop

Because `<Index>` is used, the `children` function receives a stable `index: number` (not `Accessor<number>`) and a reactive `item: Accessor<T>`:

```ts
// LazyRow children signature:
children: (item: Accessor<T[number]>, index: number) => JSX.Element
```

The `index` is NOT reactive — it reflects the item's position in the slice and won't change for a given item node. This differs from Virtual, where both `item` and `index` are accessors.

## Contrast with Virtual

Since [[VirtualRow and VirtualColumn render a sliding window over large item arrays]], Virtual uses `<List>` (from `@solid-primitives/list`) which handles the windowed slice. The slice itself changes on navigation, so `<List>` is more appropriate there. Lazy's offset only grows, never shrinks or reorders, making `<Index>` the right choice.

---

Source: [[primitives-Lazy]]
Domains:
- [[performance guide]]
- [[components guide]]
