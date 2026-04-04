# Input Throttling

> Global throttle (same key only, keyup never throttled) vs per-element throttle (all keys, returns true stopping propagation).

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

There are two distinct throttle mechanisms:

**Global throttle** (`Config.throttleInput`):
- Set in milliseconds via `Config.throttleInput`.
- Repeated presses of the SAME key within this window are discarded.
- Only applies to `keydown` events — key-up events are never globally throttled (the `!isUp` check in the source).
- Different keys are NOT throttled against each other.

**Per-element throttle** (`elm.throttleInput`):
- Set on individual `ElementNode` instances.
- Blocks ALL key events (not just the same key) through that element if the same key was recently handled there.
- Applies in BOTH capture and bubble phases.
- A throttled event in the per-element check returns `true` (event considered "handled"), which stops further propagation.
- This means a throttle on a parent can silently swallow events that children would have handled.

Throttle check happens before handler execution in both phases:

```
CAPTURE PHASE (root -> leaf):
  For each element:
    Per-element throttle check
    Check onCapture{Event} handler
    ...

BUBBLE PHASE (leaf -> root):
  For each element:
    Per-element throttle check
    Check on{Event} handler
    ...
```

## Code Example

```tsx
<view throttleInput={100} onDown={handleNavigation('down')}>
  {/* rapid down-arrow presses throttled to 100ms */}
</view>
```

## Gotchas

- Global throttle only throttles the same key — pressing Left then Right rapidly is never throttled by the global setting.
- Key-up events are NEVER globally throttled.
- Per-element throttle returns `true`, stopping propagation entirely. A throttled parent in capture phase blocks the event from reaching children.
- Per-element throttle applies to all keys through that element, not just the specific directional key the handler is for.

## Related Notes

- [focus/two-phase-propagation.md] -- throttle checks occur within the phase traversal loop
- [focus/key-handler-return-true.md] -- throttled per-element events return true, stopping propagation
