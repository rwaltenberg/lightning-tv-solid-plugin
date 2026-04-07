---
description: Calling setActiveElement(elm) rebuilds the focus path, adds the focusStateKey to newly focused ancestors, and calls onFocus/onBlur/onFocusChanged on affected elements
type: api
module: core
created: 2026-04-07
---

# setActiveElement updates focus path and fires onFocus and onBlur callbacks

`setActiveElement` is the programmatic way to move focus. It is a no-op when called with the currently active element.

When the target element differs from `activeElement`:
1. Rebuilds the focus path by walking parent links from the new element to the root.
2. For each element **entering** focus (not previously in the focus path):
   - Adds `Config.focusStateKey` to `elm.states` — this triggers the focus CSS state variant.
   - Calls `elm.onFocus(currentFocusedElm, prevFocusedElm, elm)` if defined.
   - Calls `elm.onFocusChanged(true, currentFocusedElm, prevFocusedElm, elm)` if defined.
3. For each element **leaving** focus (was in old path, absent from new path):
   - Removes `Config.focusStateKey` from `elm.states`.
   - Calls `elm.onBlur(currentFocusedElm, prevFocusedElm, elm)` if defined.
   - Calls `elm.onFocusChanged(false, currentFocusedElm, prevFocusedElm, elm)` if defined.
4. Updates the module-level `activeElement`.
5. Calls `Config.setActiveElement(elm)` — the framework hook that lets Solid signals and refs react to focus changes.

The `onFocusChanged` callback is a unified alternative to `onFocus`/`onBlur`; the first argument is a boolean indicating gained (`true`) or lost (`false`) focus, making it easier to use a single handler for both states.

```typescript
// Example element callbacks
const myEl = {
  onFocus(currentFocused, prevFocused, self) { /* gained focus */ },
  onBlur(currentFocused, prevFocused, self) { /* lost focus */ },
  onFocusChanged(hasFocus, currentFocused, prevFocused, self) { /* either */ },
};
```

Since [[focus path is an ordered array from focused leaf to root ancestor]], elements that are common ancestors of both old and new focus targets are re-evaluated: the code only adds `focusStateKey` if the element does not already have it (or is the exact newly focused element), preventing redundant callbacks on stable ancestors. Because [[focus state key is automatically added and removed from elm states during focus path changes]], navigating between children of the same Row only fires `onFocus`/`onBlur` on the children — the Row itself receives no redundant callback.

---

Related Entries:
- [[setFocus defers focus assignment via microtask to allow children to render first]] — the public API that calls setActiveElement after deferring to allow Row/Column children to render
- [[Config singleton holds all runtime Lightning configuration]] — provides the `setActiveElement` hook that connects the focus manager to SolidJS reactivity

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
