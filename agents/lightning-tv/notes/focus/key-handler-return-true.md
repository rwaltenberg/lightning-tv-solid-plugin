# Key Handlers Must Return true to Stop Propagation

> Returning false, void, or undefined from a key handler does NOT stop propagation. Only returning true stops the event.

**Source**: `spatial-navigation.md` | **Severity**: critical

## Detail

Key handlers must return `true` to stop propagation. Returning `false` is the same as returning `void` — the event continues bubbling. If a handler performs a side effect but does not return `true`, the event will also be handled by ancestors, likely causing double-navigation.

The return type is defined as:

```ts
type KeyHandlerReturn = boolean | void;
```

Returning `true` stops propagation in the current phase (capture or bubble). If no handler in the entire propagation returns `true`, the event is considered unhandled and falls through.

The `onKeyPress` fallback handler is explicitly for catching unhandled events — it fires only if no specific mapped handler (e.g., `onLeft`, `onEnter`) returns `true` first. If `onLeft` returns `true`, `onKeyPress` will never see left-arrow events on the same element.

## Code Example

```tsx
<view
  onEnter={(e, elm, focusedElm) => {
    doSomething();
    return true; // consume the event
  }}
  onBack={() => {
    navigateBack();
    return true;
  }}
/>

// Catch-all — returning void/false lets it continue bubbling
<view
  onKeyPress={(e, mappedEvent, elm, focusedElm) => {
    console.log('Unhandled key:', mappedEvent || e.key);
    // returning void/false lets it continue bubbling
  }}
/>
```

## Gotchas

- Returning `false` is semantically identical to returning `void` — the event keeps propagating. This is a common mistake for developers who expect `false` to mean "stop propagation" (as in DOM `addEventListener`).
- Double-navigation is the typical symptom: the handler fires its side effect, but the event also propagates to an ancestor that also handles it.
- `onKeyPress` only fires if no specific handler returns `true`. Do not rely on `onKeyPress` for events handleable by specific handlers.

## Related Notes

- [focus/two-phase-propagation.md] -- the full two-phase model where return true stops propagation
- [focus/handler-signature-differences.md] -- exact parameter signatures for all handler types
