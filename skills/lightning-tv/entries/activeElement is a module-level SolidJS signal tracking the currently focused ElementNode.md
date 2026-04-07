---
description: A createSignal pair exported as [activeElement, setActiveElement]; setActiveElement is injected into Config so the focus manager can drive Solid reactivity on focus changes
type: architecture
module: core
created: 2026-04-07
---

# activeElement is a module-level SolidJS signal tracking the currently focused ElementNode

The `activeElement` signal is defined in `activeElement.ts` as a simple SolidJS signal pair:

```ts
export const [activeElement, setActiveElement] = createSignal<
  ElementNode | undefined
>(undefined);
```

The signal starts as `undefined` (nothing focused). When focus changes, `setActiveElement` is called with the newly focused `ElementNode`.

The key architectural detail is how `setActiveElement` reaches the focus manager. In `createRenderer()`:

```ts
Config.setActiveElement = setActiveElement;
```

The focus manager in `core/focusManager.ts` calls `Config.setActiveElement` when focus transitions. This avoids a circular dependency — the core package doesn't import from the Solid wrapper, but the Solid wrapper injects itself into the config.

Since [[task queue suspends on focus change and resumes when renderer is idle]], the `activeElement()` signal is also what the task queue tracks to pause background work on navigation events. Any component that reads `activeElement()` in a reactive context will re-run when focus changes.

The signal is re-exported via `index.ts` making it part of the public API. Applications use it to respond to focus changes without subscribing to specific component events. This is particularly useful for Row/Column containers that want to react to focus entering their subtree without adding `onFocus` to every child — since [[focus path is an ordered array from focused leaf to root ancestor]], reading `activeElement()` and checking whether it's a descendant is a common reactive pattern.

---

Related Entries:
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — the setter side of this signal pair; called by the focus manager on every navigation event
- [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]] — the function that triggers setActiveElement on every keypress

Source: [[render]]
Domains:
- [[core guide]]
- [[focus guide]]
