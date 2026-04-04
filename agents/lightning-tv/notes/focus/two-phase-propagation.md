# Two-Phase Key Event Propagation

> Key events follow capture (root-to-leaf) then bubble (leaf-to-root). Any handler returning true stops propagation in its phase.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

Key events follow a two-phase propagation model inspired by the DOM:

```
Phase 1: CAPTURE  (root -> leaf)
Phase 2: BUBBLE   (leaf -> root)
```

The focus path array is traversed in reverse (root-to-leaf) for capture, then forward (leaf-to-root) for bubble. Any handler returning `true` stops propagation in its phase. If no handler returns `true`, the event is considered unhandled and falls through.

The full traversal sequence:

```
KeyboardEvent (keydown or keyup)
  |
  +-> Global throttle check (Config.throttleInput)
  |     THROTTLED -> return false (event discarded silently)
  |
  +-> Key hold check (for keydown only)
  |     IN HOLD MAP -> Start timeout, return (no immediate propagation)
  |
  +-> CAPTURE PHASE (root -> leaf traversal of focusPath)
  |     For each element:
  |       Per-element throttle check
  |       Check onCapture{Event} handler
  |       Check onCaptureKey / onCaptureKeyRelease handler
  |       If handler returns true -> STOP, return true
  |
  +-> BUBBLE PHASE (leaf -> root traversal of focusPath)
        For each element:
          Per-element throttle check
          Check on{Event} handler (e.g., onLeft, onEnter)
          If not handled, check onKeyPress / onKeyHold fallback
          If any handler returns true -> STOP, return true
```

Per-element throttle (`elm.throttleInput`) is checked in both phases. A throttled event returns `true` (event considered "handled"), which stops further propagation — meaning a throttle on a parent can silently swallow events that children would have handled.

For the default key map, the following handler properties are available:

**Bubble phase (leaf-to-root):**
- `onLeft`, `onRight`, `onUp`, `onDown`
- `onEnter`, `onLast`, `onSpace`, `onBack`, `onEscape`
- `onLeftRelease`, `onRightRelease`, `onUpRelease`, `onDownRelease`
- `onEnterRelease`, `onLastRelease`, `onSpaceRelease`, `onBackRelease`, `onEscapeRelease`

**Capture phase (root-to-leaf):**
- `onCaptureLeft`, `onCaptureRight`, `onCaptureUp`, `onCaptureDown`
- `onCaptureEnter`, `onCaptureLast`, `onCaptureSpace`, `onCaptureBack`, `onCaptureEscape`
- `onCaptureKey` (catch-all for any keydown in capture phase)
- `onCaptureKeyRelease` (catch-all for any keyup in capture phase)

**Fallback handlers (bubble phase only, fire if no specific key handler returns `true`):**
- `onKeyPress` -- catches any unhandled keydown
- `onKeyHold` -- catches any unhandled key-hold event

Handler properties are derived dynamically from the key map via the `EventHandlers<Map>` type:

```ts
type EventHandlers<Map> = {
  [K in keyof Map as `on${Capitalize<string & K>}`]?: KeyHandler;
} & {
  [K in keyof Map as `on${Capitalize<string & K>}Release`]?: KeyHandler;
} & {
  [K in keyof Map as `onCapture${Capitalize<string & K>}`]?: KeyHandler;
} & {
  onCaptureKey?: KeyHandler;
  onCaptureKeyRelease?: KeyHandler;
};
```

## Gotchas

- `onKeyPress` and `onKeyHold` are bubble-phase fallbacks only — they only fire if no specific handler returned `true` first.
- Per-element throttle returns `true` when throttling, which stops propagation entirely (children won't see the event if the throttle is on a parent ancestor in capture phase, or a parent in bubble phase won't see it if child throttles in bubble).
- Key-up events are never globally throttled (only keydown is subject to `Config.throttleInput`).

## Related Notes

- [focus/key-handler-return-true.md] -- returning true is what stops propagation
- [focus/handler-signature-differences.md] -- exact signatures for specific vs fallback handlers
- [focus/input-throttling.md] -- global and per-element throttle behavior
- [focus/default-key-map.md] -- the key map that determines which handler names exist
