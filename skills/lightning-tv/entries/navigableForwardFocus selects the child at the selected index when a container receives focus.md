---
description: navigableForwardFocus reads el.selected, finds the first non-skipFocus child at or after that index, updates selected, and calls setFocus on the child
type: api
module: primitives
created: 2026-04-07
---

# navigableForwardFocus selects the child at the selected index when a container receives focus

`navigableForwardFocus` is the `ForwardFocusHandler` used by [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]]. It runs when a Row/Column itself receives focus — typically when focus moves from a parent container into the list.

```ts
export const navigableForwardFocus: ForwardFocusHandler = function () {
  const navigable = this as NavigableElement;
  let selected = Math.max(navigable.selected, 0);  // floor at 0

  if (this.children.length === 0) return false;

  // clamp to valid range
  selected = clamp(selected, 0, this.children.length - 1);

  // skip over skipFocus children
  selected = findFirstFocusableChildIdx(navigable, selected);
  navigable.selected = selected;  // update selected in case skipFocus redirected
  return selectChild(navigable, selected);
};
```

Key behaviors:
- Negative `selected` values are clamped to 0
- If `selected` points to a `skipFocus` child, the next non-skipped child (in forward direction) is selected instead
- `navigable.selected` is updated as a side effect — even if no movement was intended, the property reflects the actual focused child
- Returns `false` when the container has no children (focus does not consume the event)
- `onSelectedChanged` fires via `selectChild` — even on the first focus-in

**Custom forward focus:** To forward focus to the closest element instead of the remembered index, use `spatialForwardFocus` from the same module — see [[spatialForwardFocus selects the geometrically closest child when a container receives focus]].

Since [[setFocus defers focus assignment via microtask to allow children to render first]], `navigableForwardFocus` can safely call `setFocus` on the selected child immediately — the microtask deferral ensures the child is rendered before focus actually transfers. Since [[plinko prop transfers the current row selected index to the next row on vertical navigation]], plinko runs before `navigableForwardFocus` so the `selected` property is already updated to the transferred index by the time forward focus reads it.

```tsx
<view
  forwardFocus={navigableForwardFocus}
  selected={0}
  onSelectedChanged={(idx, el, child, lastIdx) => { ... }}
>
```

---

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
- [[focus guide]]
