---
description: The primitives useFocusManager bridges core focus management into Solid.js by capturing getOwner() and wrapping setActiveElement and key handlers in runWithOwner
type: architecture
module: primitives
created: 2026-04-07
---

# primitives useFocusManager wraps core focus manager with Solid reactive owner context

The `useFocusManager` in `src/primitives/useFocusManager.ts` is a thin adapter over `core/focusManager.ts`. It does no focus logic itself — its sole job is making the core focus manager Solid.js-aware.

```typescript
export const useFocusManager = (
  userKeyMap?: Partial<KeyMap>,
  keyHoldOptions?: KeyHoldOptions,
) => {
  const owner = getOwner();
  const ownerContext = runWithOwner.bind(this, owner);
  Config.setActiveElement = (activeElm) =>
    ownerContext(() => setActiveElement(activeElm));

  const { cleanup, focusPath: focusPathCore } = useFocusManagerCore({
    userKeyMap,
    keyHoldOptions,
    ownerContext,
  });

  createEffect(
    on(activeElement, () => {
      setFocusPath([...focusPathCore()]);
    }, { defer: true }),
  );

  onCleanup(cleanup);
};
```

The three things it does:

1. **Owner capture** — `getOwner()` captures the reactive owner at call time. All subsequent key handler invocations are wrapped with `runWithOwner(owner, ...)` via `ownerContext`. This ensures reactive effects created inside key handlers are properly tracked and disposed.

2. **Config hook** — Overrides `Config.setActiveElement` to run within the owner context. The core calls this after updating `activeElement`, triggering any Solid signal updates that need reactive ownership.

3. **Reactive focusPath sync** — A `createEffect` on the `activeElement` signal keeps the exported module-level `focusPath` signal in sync with the core's focus path. The `{ defer: true }` prevents this effect from firing synchronously during setup.

Since [[ownerContext option in useFocusManager wraps key handlers for reactive framework compatibility]], passing `ownerContext` to the core ensures all propagation logic runs under Solid's reactive system.

Since [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]], the core handles the DOM listener setup; the primitives layer only adds reactivity wiring.

The exported `focusPath` signal is the primary integration point for accessibility — since [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]], the Announcer reads from this signal to know which elements entered focus and what to speak.

---

Source: [[primitives-useFocusManager]]
Domains:
- [[input guide]]
- [[accessibility guide]]
