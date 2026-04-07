---
description: Setting wrap=true on Row or Column makes navigation wrap from the last child back to the first and vice versa; all-skipFocus children with wrap cause an infinite loop
type: api
module: primitives
created: 2026-04-07
---

# NavigableProps wrap prop enables circular navigation within Row or Column children

The `wrap` prop on `NavigableProps` (used by [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]]) enables circular navigation. Pressing the next-direction key at the last child wraps back to the first focusable child, and pressing the previous-direction key at the first child wraps to the last focusable child.

```tsx
<Row wrap={true}>
  <Item />  {/* pressing Left from here focuses last item */}
  <Item />
  <Item />  {/* pressing Right from here focuses first item */}
</Row>
```

The wrap behavior is implemented in `findFirstFocusableChildIdx`:

```ts
function findFirstFocusableChildIdx(el, from = 0, delta = 1) {
  for (let i = from; ; i += delta) {
    if (!idxInArray(i, el.children)) {
      if (el.wrap) {
        i = (i + el.children.length) % el.children.length;
      } else break;  // without wrap, stop at boundary
    }
    if (!el.children[i]?.skipFocus) return i;
  }
  return -1;
}
```

**Critical gotcha:** The loop runs indefinitely when `wrap=true`. If ALL children have `skipFocus=true`, this loop never terminates and hangs the application. Always ensure at least one child is focusable when using `wrap`.

**Contrast with Grid `looping`:** Grid uses a separate `looping` prop and implements its own wrap logic that preserves column alignment. Row/Column `wrap` is purely sequential — see [[Grid looping wraps to the same column in the first or last row not to item index 0]].

Since [[moveSelection skips skipFocus children and supports plinko index transfer between rows]], `wrap` is read inside `findFirstFocusableChildIdx` during `moveSelection` — it's not a separate code path but an integrated part of the traversal.

---

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
