---
description: Row and Column chain their scroll function into onLayout only when the selected prop is truthy at mount, so selected={0} never triggers an initial scroll
type: gotcha
module: primitives
created: 2026-04-07
---

# Row onLayout only chains scrollRow when selected prop is non-zero at mount

Row and Column have this conditional in their render:

```tsx
onLayout={
  /* @once */
  props.selected
    ? chainFunctions(props.onLayout, scrollRow)
    : props.onLayout
}
```

`props.selected` is evaluated as a boolean. Because JavaScript treats `0` as falsy:
- `selected={0}` — `scrollRow` is NOT chained into onLayout
- `selected={5}` — `scrollRow` IS chained, so when layout completes, the list positions itself at item 5

This means if you mount a Row with `selected={0}` (or no `selected` prop, which defaults to `0`), no initial scroll occurs and the list starts at position 0. This is almost always the desired behavior.

However, if you mount with `selected={5}`, the list will scroll to item 5 immediately after layout. This is how Row supports an initial focused item that isn't the first.

Since [[Row and Column use @once annotations so handler props cannot be updated after mount]], this condition is evaluated exactly once and cannot be changed reactively. If `selected` becomes non-zero later via a signal, no compensating scroll happens.

**Pattern:** To start at a non-zero item, pass `selected` as a static value at mount time, not a reactive signal that starts at 0 and changes.

---

Source: [[primitives-Row]]
Domains:
- [[navigation guide]]
