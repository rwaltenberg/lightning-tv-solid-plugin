---
description: scrollIndex sets the item index at which auto-mode scrolling begins, creating a fixed zone at the start where focus moves without the list shifting
type: api
module: primitives
created: 2026-04-07
---

# scrollIndex prop delays auto scroll start until selected item reaches the threshold index

When `scroll="auto"` (or as default behavior), Row and Column accept a `scrollIndex` prop that creates a non-scrolling zone at the start of the list. Focus moves through the first `scrollIndex` items without the container shifting; scrolling begins only when the selected index reaches `scrollIndex`.

```tsx
<Row scroll="auto" scrollIndex={2}>
  {/* Items 0, 1, 2 stay in place while focus moves through them.
      When focus reaches item 2, the list starts scrolling */}
  <Item />
  <Item />
  <Item />
  <Item />
  <Item />
</Row>
```

The implementation in `withScrolling`:
```ts
if (scrollIndex && scrollIndex > 0) {
  if (isIncrementing && selected >= scrollIndex) {
    nextPosition = rootPosition - selectedSize - gap;
  } else if (!isIncrementing && selected < nearEndIndex) {
    nextPosition = rootPosition + selectedSize + gap;
  }
}
```
Where `nearEndIndex = totalItems - scrollIndex`. Going backward, scrolling stops again when selected returns to within `scrollIndex` of the end.

This creates symmetrical "dead zones" at the start and end: the list scrolls through the middle items, but focus moves through the bookend zones without list movement.

**Common pattern for TV UIs:** Set `scrollIndex={2}` so the first two items are always visible at the leading edge before the list starts scrolling. This ensures navigation context is always visible.

Since [[withScrolling implements six scroll modes for navigable list containers]], `scrollIndex` only applies in `auto` mode. In `edge`, `always`, `center`, or `bounded` modes, `scrollIndex` is ignored.

---

Source: [[primitives-withScrolling]]
Domains:
- [[navigation guide]]
