---
description: Keys in keyHoldMapEntries trigger a hold event after holdThreshold ms (default 500); releasing before the threshold cancels hold detection and fires a normal keypress instead
type: architecture
module: core
created: 2026-04-07
---

# key hold detection fires after 500ms and cancels if key is released early

The focus manager supports key hold detection for keys registered in `keyHoldMapEntries`. By default this map is empty (the `Enter: 'EnterHold'` entry is commented out), so hold detection is opt-in via `keyHoldOptions.userKeyHoldMap` in [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]].

## Hold detection lifecycle

**Keydown** (key is in holdMap):
1. If no timer exists for this key, start a `setTimeout` with `delay` (default `500`ms or `keyHoldOptions.holdThreshold`).
2. The timeout calls `propagateKeyPress(event, mappedHoldEvent, isHold=true)`.
3. The timer reference is replaced with `true` to mark "hold fired".
4. If a timer already exists for this key, the keydown is ignored (prevents repeat triggering on TV remotes that generate repeated keydown events while held).

**Keyup**:
- If `keyHoldTimeouts[key] === true`: hold already fired. Clean up the entry, and still fire the release propagation via `propagateKeyPress(keyup, mappedKeyEvent, false, isUp=true)`.
- If `keyHoldTimeouts[key]` is a number: the hold didn't complete. Cancel the timer. Fire a **normal** `propagateKeyPress` for the keydown event (treating it as a tap), then fire the release propagation.
- If no entry: not a held key, normal keyup handling.

## Handlers for hold events

In the bubble phase, `isHold=true` causes `fallbackHandlerKey` to be set to `'onKeyHold'` rather than `'onKeyPress'`. Specific mapped handlers (`on${mappedHoldEvent}`) also work normally.

```typescript
// Example: Register Enter as a hold key
useFocusManager({
  keyHoldOptions: {
    userKeyHoldMap: { Enter: 'EnterHold' },
    holdThreshold: 800,
  }
});

// Element handler
const el = {
  onEnterHold(e, elm, focusedElm) { /* long-press action */ return true; }
};
```

For per-component hold detection independent of the global hold map, see [[useHold distinguishes quick press from long hold using a threshold timeout and wasHeld flag]] — it implements the same timeout/cancel pattern but scoped to a single element without requiring global configuration.

---
Source: [[core-focusManager]]
Domains:
- [[focus guide]]
- [[input guide]]
