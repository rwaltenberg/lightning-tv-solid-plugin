---
description: Config.throttleInput sets a global ms window that drops repeated same-key presses before propagation; elm.throttleInput applies the same logic per-element during both capture and bubble phases
type: api
module: core
created: 2026-04-07
---

# throttleInput can be set globally on Config or per-element to suppress rapid key repeats

The focus manager has two levels of input throttling to handle TV remote key repeat behavior.

## Global throttle (Config.throttleInput)

Applied before the capture phase starts, on keydown events only. If the same key (`e.key || e.keyCode`) is seen within `Config.throttleInput` milliseconds of the last keydown, the event is dropped entirely — `propagateKeyPress` returns `false` without calling any handlers.

```typescript
if (Config.throttleInput && sameKey && currentTime - lastGlobalKeyPressTime < Config.throttleInput) {
  return false; // dropped
}
```

This is a global gate: no element sees the event at all when the global throttle fires.

## Per-element throttle (elm.throttleInput)

Applied inside both the capture and bubble loops, per element. If an element has `throttleInput` set and the same key was recently seen for that element, propagation stops at that element and returns `true` (consumed). The timestamp is tracked in `elm._lastAnyKeyPressTime`.

Per-element throttle fires on both keydown and keyup, unlike global throttle. It also applies to the capture phase, so a parent with `throttleInput` can suppress events before they reach children.

## Important gotcha

Per-element throttle returns `true` (consumed), not `false` (dropped). This means throttled events at the element level are treated as handled, which prevents further bubbling. The global throttle returns `false`, which has no side effects except skipping this keypress.

Since [[focus manager propagates key events in capture then bubble phase]], element-level throttle can occur in either phase — whichever phase reaches the throttled element first. Setting `throttleInput` on a [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] or [[Column component is a vertical navigable list with flexDirection column and 30px gap]] is a common pattern for preventing rapid navigation key repeats from queuing too many scroll animations.

---
Note that `throttleInput` and [[useHold distinguishes quick press from long hold using a threshold timeout and wasHeld flag]] can interact — if global throttle drops a keydown event that `useHold` was waiting to see as a keyup, hold detection may produce unexpected results. Per the open question in the input guide, this interaction is undocumented.

Related Entries:
- [[Config singleton holds all runtime Lightning configuration]] — Config.throttleInput sets the global gate value checked inside useFocusManager's listener

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
- [[input guide]]
