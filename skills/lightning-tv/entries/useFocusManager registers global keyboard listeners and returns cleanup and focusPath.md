---
description: useFocusManager() attaches keydown/keyup listeners to document, applies user key maps, and returns { cleanup, focusPath } — must be called once per app lifecycle
type: api
module: core
created: 2026-04-07
---

# useFocusManager registers global keyboard listeners and returns cleanup and focusPath

`useFocusManager` is the entry point for the focus/input system. It should be called once during app initialization. Calling it multiple times attaches duplicate listeners.

```typescript
const { cleanup, focusPath } = useFocusManager({
  userKeyMap?: Partial<KeyMap>,
  keyHoldOptions?: {
    userKeyHoldMap?: Partial<KeyMap>,
    holdThreshold?: number,         // ms before hold fires; default 500
  },
  ownerContext?: (cb: () => void) => void, // wraps handlers; default identity fn
});
```

**Parameters:**
- `userKeyMap` — Merges into the default key map. Values are framework event names; keys are browser key names or codes. Set a value to `null` to delete a mapping. Array values assign multiple browser keys to one event.
- `keyHoldOptions.userKeyHoldMap` — Adds keys that trigger hold events after `holdThreshold` ms.
- `keyHoldOptions.holdThreshold` — Time in ms before a held key fires the hold handler (default `500`).
- `ownerContext` — Wraps both keydown and keyup handlers. Used in frameworks like Solid.js where reactive effects need an owner context (`runWithOwner`).

**Returns:**
- `cleanup()` — Removes both `keydown` and `keyup` listeners from `document`, and cancels any in-flight hold timers. Call on component unmount or app teardown.
- `focusPath()` — Returns the current focus path array (live reference, not a snapshot).

Since [[focus manager propagates key events in capture then bubble phase]], the listeners set up by `useFocusManager` drive all event propagation through the system. Without calling this function, no key events are processed regardless of how many `onLeft`/`onEnter` handlers are registered on elements.

In Solid.js apps, always import from primitives rather than core directly — since [[primitives useFocusManager wraps core focus manager with Solid reactive owner context]], the primitives version adds the reactive ownership and `focusPath` signal wiring that the core version omits.

Before `useFocusManager` is called, ensure `Config.setActiveElement` is injected (done automatically by `createRenderer`). Since [[throttleInput can be set globally on Config or per-element to suppress rapid key repeats]], the global throttle configured on `Config` is checked inside the listeners that `useFocusManager` registers.

---

Related Entries:
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — the function called on every focus change by the listeners registered here
- [[focus path is an ordered array from focused leaf to root ancestor]] — the live path array exposed via the `focusPath()` return value

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
- [[input guide]]
