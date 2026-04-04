# Handler Signature Differences: Specific vs Fallback Handlers

> Specific handlers (onLeft, onEnter) and fallback handlers (onKeyPress, onKeyHold) have different parameter orders. Mixing them up causes subtle bugs.

**Source**: `spatial-navigation.md` | **Severity**: critical

## Detail

There are two distinct handler signatures. Confusing them produces bugs where parameters contain unexpected values.

**Specific key handlers** (e.g., `onLeft`, `onEnter`, `onBack`, `onCaptureDown`):

```ts
type KeyHandlerReturn = boolean | void;

type KeyHandler = (
  this: ElementNode,
  e: KeyboardEvent,
  target: ElementNode,       // the element that owns this handler
  handlerElm: ElementNode,   // the leaf-focused element
  mappedEvent?: string,       // the mapped event name (only for capture handlers)
) => KeyHandlerReturn;
```

**Generic fallback handlers** (`onKeyPress`, `onKeyHold`) — NOTE: parameter order differs from `KeyHandler`:

```ts
onKeyPress(
  this: ElementNode,
  e: KeyboardEvent,
  mappedKeyEvent: string | undefined,
  handlerElm: ElementNode,
  currentFocusedElm: ElementNode,
) => KeyHandlerReturn;
```

Key differences:
- In `KeyHandler`, the second parameter is `target` (the element owning the handler), and the third is `handlerElm` (the leaf-focused element).
- In `onKeyPress`/`onKeyHold`, the second parameter is `mappedKeyEvent` (the mapped string event name), and `handlerElm` and `currentFocusedElm` are third and fourth.
- `mappedEvent` in `KeyHandler` is optional and only present for capture handlers.

`onKeyPress` is a fallback handler — it only fires if no specific mapped handler (e.g., `onLeft`, `onEnter`) returns `true` first.

`onFocus`, `onBlur`, and `onFocusChanged` have their own distinct signatures:

```ts
interface FocusNode {
  onFocus?: (
    this: ElementNode,
    currentFocusedElm: ElementNode,
    prevFocusedElm: ElementNode | undefined,
    nodeWithCallback: ElementNode,
  ) => void;

  onFocusChanged?: (
    this: ElementNode,
    hasFocus: boolean,
    currentFocusedElm: ElementNode,
    prevFocusedElm: ElementNode | undefined,
    nodeWithCallback: ElementNode,
  ) => void;

  onBlur?: (
    this: ElementNode,
    currentFocusedElm: ElementNode,
    prevFocusedElm: ElementNode,
    nodeWithCallback: ElementNode,
  ) => void;
}
```

All three `FocusNode` handlers receive `nodeWithCallback` — the specific node in the path that owns the handler, which may be an ancestor of the actual leaf.

## Code Example

```tsx
// Specific handler — (e, target, handlerElm)
<view
  onEnter={(e, elm, focusedElm) => {
    doSomething();
    return true;
  }}
/>

// Fallback handler — (e, mappedEvent, elm, focusedElm)
<view
  onKeyPress={(e, mappedEvent, elm, focusedElm) => {
    console.log('Unhandled key:', mappedEvent || e.key);
  }}
/>
```

## Gotchas

- `onKeyPress` and `onKeyHold` have `mappedKeyEvent` as the second parameter, NOT `target`. Treating it as a `KeyHandler` will silently misuse the event name string as if it were an `ElementNode`.
- `onFocusChanged` receives `hasFocus: boolean` as its first parameter after `this`, making its signature different from both `onFocus` and `onBlur`.
- `nodeWithCallback` in focus handlers is the ancestor that owns the callback, not necessarily the currently focused leaf.

## Related Notes

- [focus/key-handler-return-true.md] -- return true semantics apply to all key handler types
- [focus/two-phase-propagation.md] -- when each handler type fires
