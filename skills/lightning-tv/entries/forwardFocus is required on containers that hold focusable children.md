---
description: Containers without forwardFocus never route focus to children; the renderer calls forwardFocus when a non-leaf node receives focus
type: gotcha
module: core
created: 2026-04-07
---

# forwardFocus is required on containers that hold focusable children

When a non-leaf element receives focus, the renderer calls its `forwardFocus` handler to determine which child should actually become active. Without `forwardFocus`, the container becomes a focus dead-end — `setFocus()` lands on the container itself and children are never reached.

## How forwardFocus works

`forwardFocus` is typed as `ForwardFocusHandler`, a function invoked with `this` bound to the container node. It must return `true` (successfully forwarded) or `false` (could not forward). When it returns `false`, focus stays on the calling node.

The framework ships two ready-made handlers:

```typescript
// Standard Row/Column handler — selects the child at `selected` index
export const navigableForwardFocus: lng.ForwardFocusHandler = function () {
  const navigable = this as lngp.NavigableElement;

  let selected = Math.max(navigable.selected, 0);

  if (this.children.length === 0) {
    return false;
  }
  // clamps selected, skips skipFocus children, then calls child.setFocus()
  selected = findFirstFocusableChildIdx(navigable, selected);
  navigable.selected = selected;
  return selectChild(navigable, selected);
};

// Spatial handler — selects geometrically closest child to previous active element
export const spatialForwardFocus: lng.ForwardFocusHandler = function () { ... };
```

Both Row and Column automatically wire `forwardFocus={navigableForwardFocus}`. Custom container views must do this explicitly:

```tsx
// Missing forwardFocus — focus never reaches children
<view onRight={handleRight}>
  <ChildItem />
  <ChildItem />
</view>

// Correct — focus forwards to first focusable child at selected index
<view
  selected={0}
  forwardFocus={navigableForwardFocus}
  onRight={handleRight}
>
  <ChildItem />
  <ChildItem />
</view>
```

## Common symptom

If your container appears focused (has the focus state) but key presses don't work and children never receive `$focus` style, the container is missing `forwardFocus`. The key handlers on children never fire because focus never left the parent.

## When returning false is correct

A container may intentionally return `false` from `forwardFocus` when it has no children yet (e.g., async loading). The framework will leave focus on the container. Once children arrive, call `setFocus()` again on the container to trigger another forward pass.

---

Related Entries:
- [[navigableForwardFocus selects the child at the selected index when a container receives focus]] — the standard implementation used by Row and Column
- [[spatialForwardFocus selects the geometrically closest child when a container receives focus]] — geometric alternative for grid-like containers
- [[focus manager propagates key events in capture then bubble phase]] — the broader event flow that forwardFocus feeds into
- [[setFocus defers focus assignment via microtask to allow children to render first]] — how focus is actually assigned after forwardFocus returns true

Domains:
- [[focus guide]]
- [[navigation guide]]
