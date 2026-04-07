---
description: moveSelection is the core selection engine for Row/Column — advances by delta, skips skipFocus children, and copies selected index across rows when plinko is enabled
type: architecture
module: primitives
created: 2026-04-07
---

# moveSelection skips skipFocus children and supports plinko index transfer between rows

`moveSelection(el, delta)` is the internal function called by all directional key handlers in Row and Column. It finds the next focusable child in the given direction and handles two special behaviors: `skipFocus` and `plinko`.

```ts
export function moveSelection(el: NavigableElement, delta: number): boolean {
  let selected = findFirstFocusableChildIdx(el, el.selected + delta, delta);

  if (selected === -1) {
    // no focusable child found in that direction
    if (!idxInArray(el.selected, el.children) || el.children[el.selected]?.skipFocus || isFocused(el.children[el.selected]!)) {
      return false; // propagate key event up
    }
    selected = el.selected; // stay on current
  }

  if (el.plinko) {
    const lastSelectedChild = el.children[el.selected];
    const num = lastSelectedChild.selected || 0;
    active.selected = num < active.children.length ? num : active.children.length - 1;
  }

  return selectChild(el, selected);
}
```

## skipFocus

Any child with `skipFocus=true` is invisible to navigation. `findFirstFocusableChildIdx` skans past them. If all remaining children in the direction have `skipFocus=true`, `moveSelection` returns `false`, allowing the key event to bubble up to the parent.

```tsx
<Row>
  <Item />
  <Item skipFocus={true} />  {/* navigation skips this */}
  <Item />
</Row>
```

When `el.wrap=true`, `findFirstFocusableChildIdx` wraps around the array. If ALL children have `skipFocus=true`, this causes an infinite loop — always ensure at least one focusable child when using `wrap`.

## plinko

When `el.plinko=true`, before selecting the target child, `moveSelection` copies the CURRENT child's `selected` value to the target child:

```ts
const num = lastSelectedChild.selected || 0;
active.selected = num < active.children.length ? num : active.children.length - 1;
```

This is designed for Column-of-Rows layouts: when moving up/down between Row children, the next Row starts with its selected item at the same column index the previous Row was at. The result is that focus moves "straight up/down" through a grid structure.

**Plinko requires nested navigable elements.** If children don't have a `selected` property, the transfer has no effect.

```tsx
<Column plinko={true}>
  <Row>{/* items */}</Row>  {/* selected index is transferred here */}
  <Row>{/* items */}</Row>
  <Row>{/* items */}</Row>
</Column>
```

## Return value

Returns `true` if selection moved, `false` if the key event should bubble up. Row/Column propagate this return value, so returning `false` allows a parent container to handle the key. This follows the same contract as [[KeyHandlerReturn true stops key event propagation up the focus chain]] — returning `true` means the event was consumed by this navigable list and will not continue up the [[focus path is an ordered array from focused leaf to root ancestor]].

---

Related Entries:
- [[navigableForwardFocus selects the child at the selected index when a container receives focus]] — called via selectChild after moveSelection finds the destination; plinko must run before this to ensure the right index is read
- [[Row and Column apply direction-specific transitions on each navigation key press]] — handleNavigation calls moveSelection after setting the directional transition

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
