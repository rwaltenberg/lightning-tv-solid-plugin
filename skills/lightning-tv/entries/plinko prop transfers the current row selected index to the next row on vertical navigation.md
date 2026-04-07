---
description: plinko=true on a Column causes moveSelection to copy the departing row's selected index to the arriving row before focusing, enabling straight-line vertical navigation through nested rows
type: pattern
module: primitives
created: 2026-04-07
---

# plinko prop transfers the current row selected index to the next row on vertical navigation

`plinko` is a prop on `NavigableProps` designed for Column-of-Rows layouts. When a user moves up or down through rows, plinko ensures the next row starts focused at the same column position the current row was at.

```tsx
<Column plinko={true}>
  <Row>  {/* if selected=2 here when moving down... */}
    <Item /><Item /><Item /><Item />
  </Row>
  <Row>  {/* ...this Row starts with selected=2 */}
    <Item /><Item /><Item /><Item />
  </Row>
</Column>
```

The mechanism in `moveSelection`:
```ts
if (el.plinko) {
  const lastSelectedChild = el.children[el.selected];
  const num = lastSelectedChild.selected || 0;
  active.selected = num < active.children.length
    ? num
    : active.children.length - 1;
}
```

Before calling `selectChild` on the destination row, `moveSelection` reads `lastSelectedChild.selected` (the current row's selected index) and assigns it to `active.selected` (the next row). The next row's `navigableForwardFocus` then focuses child at that index.

**Requirements:**
- The Column must have `plinko={true}`
- Children must be navigable elements (Rows or similar) with a `selected` property
- The destination child must have enough children to accommodate the index (clamped to `child.children.length - 1`)

**Does not work with Grid children:** Grid manages its own `focusedIndex` signal internally. Assigning `active.selected` won't affect a Grid child because Grid reads from its internal signal, not the `selected` prop reactively after mount.

Since [[moveSelection skips skipFocus children and supports plinko index transfer between rows]], plinko executes only when `moveSelection` finds a valid destination. If the next item is the same (edge case), plinko still runs. Because [[navigableForwardFocus selects the child at the selected index when a container receives focus]] reads `navigable.selected` when a Row receives focus, plinko must update `active.selected` BEFORE focus transfers — `moveSelection` handles this ordering correctly.

---

Related Entries:
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — plinko is typically set on the Column, not the Row children
- [[NavigableProps plinko syncs selected index between successive navigable containers]] — the prop-level type definition for this behavior

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
