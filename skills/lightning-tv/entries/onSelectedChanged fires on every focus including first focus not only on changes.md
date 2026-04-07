---
description: selectChild always calls onSelectedChanged when a child is focused, including the initial focus-in, not just when the selected index actually changes
type: gotcha
module: primitives
created: 2026-04-07
---

# onSelectedChanged fires on every focus including first focus not only on changes

The `selectChild` function called by all navigable elements explicitly fires `onSelectedChanged` on every focus event, including the initial one:

```ts
function selectChild(el: NavigableElement, index: number): boolean {
  // ...
  el.selected = index;
  if (!lng.isFocused(child)) child.setFocus();
  // Always call onSelectedChanged on first focus for clients
  el.onSelectedChanged?.(index, el, child as ElementNode, lastSelected);
  return true;
}
```

The comment in the source confirms this is intentional: "Always call onSelectedChanged on first focus for clients."

**Implications:**

1. When a Row/Column first receives focus (via `navigableForwardFocus`), `onSelectedChanged` fires with `(currentIndex, container, child, lastSelected)` even if the item was already selected.

2. `lastSelected` is available — on first focus it will be the value of `el.selected` before the `selectChild` call, which may be the same as the new index.

3. You cannot use `index !== lastSelected` as a guard to detect "real" changes because both values may be equal on first focus-in.

**Pattern:** To distinguish first focus from actual navigation, track focus state separately or compare within `onFocus`/`onBlur` handlers rather than relying on `onSelectedChanged` alone.

Since [[navigableForwardFocus selects the child at the selected index when a container receives focus]], the chain is: container focus → `navigableForwardFocus` → `selectChild` → `onSelectedChanged`.

---

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
- [[focus guide]]
