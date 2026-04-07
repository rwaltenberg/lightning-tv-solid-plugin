---
description: When a clicked element has neither onMouseClick nor onEnter, useMouse calls setFocus() then dispatches a real Enter keydown+keyup event to the document, routing through the normal keyboard propagation system
type: gotcha
module: primitives
created: 2026-04-07
---

# useMouse click falls back to synthetic Enter keyboard event when element has no explicit click handler

When `useMouse` handles a click and the target element has no `onMouseClick` or `onEnter` handler, it dispatches a synthetic keyboard event instead of calling a handler directly:

```typescript
clickedElement.setFocus();
setTimeout(() => {
  document.dispatchEvent(createKeyboardEvent('Enter', 13));
  setTimeout(
    () => document.body.dispatchEvent(createKeyboardEvent('Enter', 13, 'keyup')),
    1,
  );
}, 1);
```

The 1ms `setTimeout` allows `setFocus()` to complete (it defers via microtask) before the Enter event fires. The second 1ms timeout sends the `keyup` so that any hold-release logic resolves correctly.

## Why This Matters

This means clicking an element that relies only on keyboard Enter behavior will work correctly via mouse — you do not need to add `onMouseClick` handlers to every element. The click transparently becomes a keyboard Enter through the normal focus + propagation system.

## Priority Order

1. `onMouseClick(event, element)` — called with mouse event and element reference
2. `onEnter()` — called with no arguments (simpler signature)
3. Synthetic Enter — `setFocus()` + real keyboard event dispatched to document

## Gotcha: `onMouseClick` vs `onEnter` Signature Difference

`onMouseClick` receives `(MouseEvent, ElementNode)` as arguments. `onEnter` receives no arguments. If you need the mouse event data (e.g., click coordinates), you must use `onMouseClick`.

## Gotcha: Synthetic Event Target

The synthetic `Enter` keydown is dispatched to `document`, not `document.body`. The keyup is dispatched to `document.body`. This asymmetry exists in the original code and matches how the focus manager's keyboard listener is attached (to `document`).

Since [[useMouse adds pointer support via BFS hit testing over the Lightning element tree]] describes the full click flow, this entry covers the fallback behavior specifically.

---

Source: [[primitives-useMouse]]
Domains:
- [[input guide]]
