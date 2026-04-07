---
description: When UseMouseOptions.customStates is provided, useMouse uses dollar-prefix custom states for hover and pressed feedback instead of calling setFocus, enabling non-focus pointer interactions
type: pattern
module: primitives
created: 2026-04-07
---

# useMouse custom states mode separates hover and pressed visual feedback from focus management

By default, `useMouse` calls `element.setFocus()` when the cursor hovers over a focusable element. Custom states mode decouples hover/pressed visual feedback from the focus system — useful when TV and mouse behaviors need to coexist independently.

```typescript
useMouse(rootNode, 100, {
  customStates: {
    hoverState: '$hover',    // applied while cursor is over element
    pressedState: '$pressed', // applied while mouse button is held
  }
});
```

`CustomState` is enforced as `\`$\${string}\`` — the type system ensures dollar-prefix naming consistent with [[dollar-prefix state keys in NodeStyles apply style variants based on active states]].

## State Lifecycle

**Hover state:**
- Added to element when cursor enters its bounds
- Removed from previous element when cursor moves to a new one
- Removed from current element when cursor leaves all interactive elements

**Pressed state:**
- Added on `mousedown` (to the element currently under cursor with `hoverState`)
- Removed on `click` (before click handler fires)

## Exported Helpers

Three utility functions work with any `RenderableNode` (ElementNode, ElementText, or TextNode):

```typescript
addCustomStateToElement(element, '$hover');
removeCustomStateFromElement(element, '$hover');
hasCustomState(element, '$hover'); // returns boolean
```

These are exported for use in testing, manual state management, or building custom interaction systems.

## Click Behavior in Custom States Mode

When `customStates` is provided, click handling uses `findElementWithCustomState` to find the clicked element by `hoverState` rather than by active focus. This means click events reliably target whatever the cursor is over, not the currently keyboard-focused element.

## When to Use

Use custom states mode when:
- You need `$hover` style variants in your `NodeStyles` without changing keyboard focus
- Multiple pointers or overlapping interactive areas require explicit state tracking
- You want visual feedback on mousedown (`$pressed`) separate from selection

Use default (focus) mode when:
- Mouse should drive focus the same way keyboard does
- Single pointer device, simple TV-remote + mouse combo

---

Source: [[primitives-useMouse]]
Domains:
- [[input guide]]
