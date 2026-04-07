---
description: Key events travel root-to-leaf in capture phase (onCaptureKey / onCapture{Event}) then leaf-to-root in bubble phase (on{Event} / onKeyPress); returning true from any handler stops propagation
type: architecture
module: core
created: 2026-04-07
---

# focus manager propagates key events in capture then bubble phase

Every key press goes through two ordered phases before being considered unhandled.

**Capture phase** — iterates the focus path from root ancestor to leaf (high index to 0). Each element may intercept the event using:
- `onCapture{MappedEvent}` — e.g., `onCaptureLeft`, `onCaptureEnter`
- `onCaptureKey` — catches any key (keydown variant)
- `onCaptureKeyRelease` — catches any key on keyup

**Bubble phase** — iterates the focus path from leaf to root (index 0 to last). Each element may handle the event using:
- `on{MappedEvent}` — e.g., `onLeft`, `onEnter`
- `on{MappedEvent}Release` — e.g., `onLeftRelease`, for keyup
- `onKeyPress` — fallback for any unmapped key press
- `onKeyHold` — fallback for hold events

Returning `true` from any handler stops propagation immediately. Returning `undefined` or `false` allows the event to continue to the next element.

```typescript
// Capture phase (root → leaf)
for (let i = numItems - 1; i >= 0; i--) {
  const captureHandler = elm[captureEvent] || elm[captureKey];
  if (isFunction(captureHandler) && captureHandler.call(...) === true) return true;
}

// Bubble phase (leaf → root)
for (let i = 0; i < numItems; i++) {
  if (eventHandler.call(...) === true) return true;
  if (fallbackHandler.call(...) === true) return true;
}
```

Because [[focus path is an ordered array from focused leaf to root ancestor]], the direction of traversal is determined by the array index direction.

Parent containers use the capture phase to override child key handlers — for example, since [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]] register their `onLeft`/`onRight`/`onUp`/`onDown` handlers in the bubble phase, a parent container can intercept these with `onCaptureLeft` before they reach the navigable list. Children use the bubble phase for their primary interaction.

Key-up events also go through both phases, but only the release handlers (`onCaptureKeyRelease`, `on{Event}Release`) and no fallback.

---

Related Entries:
- [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]] — the entry point that drives this propagation; without calling it, no key events reach any handler
- [[KeyHandlerReturn true stops key event propagation up the focus chain]] — the return value contract that halts traversal in both phases
- [[moveSelection skips skipFocus children and supports plinko index transfer between rows]] — the Row/Column selection engine invoked through the bubble phase on directional key events

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
