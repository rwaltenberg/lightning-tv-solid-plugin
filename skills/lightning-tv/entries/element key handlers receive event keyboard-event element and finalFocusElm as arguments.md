---
description: Bubble-phase handlers are called with (e, elm, finalFocusElm); capture-phase handlers are called with (e, elm, finalFocusElm, mappedEvent); fallback handlers receive (e, mappedEvent, elm, finalFocusElm)
type: api
module: core
created: 2026-04-07
---

# element key handlers receive event keyboard-event element and finalFocusElm as arguments

Handler signatures differ slightly depending on which kind of handler is being called during [[focus manager propagates key events in capture then bubble phase]].

## Capture phase handlers

`onCapture{Event}` and `onCaptureKey`/`onCaptureKeyRelease`:
```typescript
handler(e: KeyboardEvent, elm: ElementNode, finalFocusElm: ElementNode, mappedEvent?: string): boolean | void
```
- `e` — the original browser `KeyboardEvent`
- `elm` — the element whose handler is being called (same as `this`)
- `finalFocusElm` — always `focusPath[0]`, the deepest focused element (useful for parent containers that need to know the exact focused child)
- `mappedEvent` — the framework event name (e.g., `'Left'`), or `undefined` if the key has no mapping

## Bubble phase: specific event handlers

`on{MappedEvent}` and `on{MappedEvent}Release`:
```typescript
handler(e: KeyboardEvent, elm: ElementNode, finalFocusElm: ElementNode): boolean | void
```

## Bubble phase: fallback handlers

`onKeyPress` and `onKeyHold`:
```typescript
handler(e: KeyboardEvent, mappedEvent: string | undefined, elm: ElementNode, finalFocusElm: ElementNode): boolean | void
```
Note the argument order difference: `mappedEvent` comes second (before `elm`) in fallback handlers, unlike capture handlers where it comes last.

## Common pattern

```typescript
const row = {
  onCaptureLeft(e, elm, finalFocused) {
    // Intercept Left before focused child handles it
    return this.handleNavigation('left') ? true : undefined;
  },
  onLeft(e, elm, finalFocused) {
    // Handle Left in bubble phase
    return true;
  },
  onKeyPress(e, mappedEvent, elm, finalFocused) {
    // Catch-all fallback
    console.log('Unhandled key:', mappedEvent || e.key);
  }
};
```

The `finalFocusElm` argument is especially useful in container components: a [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] parent can use it in `onCaptureLeft` to know which child item is focused before deciding whether to intercept. Since [[KeyHandlerReturn true stops key event propagation up the focus chain]], handlers that return `true` prevent the bubble phase from reaching [[moveSelection skips skipFocus children and supports plinko index transfer between rows]].

---

Related Entries:
- [[FocusNode interface defines the focus lifecycle callbacks]] — the interface that declares onKeyPress/onKeyHold, which use the fallback handler signature

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
