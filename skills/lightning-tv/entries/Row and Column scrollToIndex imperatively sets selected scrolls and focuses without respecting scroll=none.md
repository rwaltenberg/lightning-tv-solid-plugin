---
description: The scrollToIndex method exposed on Row and Column elements always calls the scroll function regardless of the scroll prop, bypassing the scroll='none' guard
type: gotcha
module: primitives
created: 2026-04-07
---

# Row and Column scrollToIndex imperatively sets selected scrolls and focuses without respecting scroll=none

Both Row and Column expose a `scrollToIndex` method on their underlying `ElementNode`. Calling it does three things in order:

```ts
function scrollToIndex(this: ElementNode, index: number) {
  this.selected = index;        // 1. update selected index
  scrollRow(index, this);       // 2. reposition the container
  this.children[index]?.setFocus(); // 3. focus the child
}
```

The critical behavior: `scrollRow`/`scrollColumn` is called unconditionally — there is no `scroll !== 'none'` check here. This differs from `onSelectedChanged`, which skips scrolling when `scroll='none'`.

**Consequence:** Setting `scroll='none'` does NOT prevent programmatic `scrollToIndex` calls from repositioning the container.

To call `scrollToIndex`:
```tsx
let rowRef!: ElementNode;
<Row ref={rowRef} scroll="none" />

// Later:
rowRef.scrollToIndex(5); // WILL scroll even though scroll="none"
```

This is useful when you want to disable reactive scrolling (focus-driven) but still support programmatic jumps. It can be surprising if you expect `scroll='none'` to lock the container position entirely.

Since [[Row and Column use @once annotations so handler props cannot be updated after mount]], the `scrollToIndex` method itself is also `@once` — but since it's a `this`-bound method on the node rather than a prop-derived function, this mostly doesn't matter. The `setFocus()` call on step 3 goes through [[setFocus defers focus assignment via microtask to allow children to render first]], so the child's focus fires asynchronously even though the scroll is immediate.

---

Related Entries:
- [[Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus]] — contrast: Grid's scrollToIndex has a different two-step focus sequence

Source: [[primitives-Row]]
Domains:
- [[navigation guide]]
