---
description: Saves the active element on route leave and restores it on return if still a descendant; otherwise focuses the route root
type: pattern
module: primitives
created: 2026-04-07
---

# KeepAliveRoute restores focused element when returning to a preserved route

`KeepAliveRoute` is the router-integrated version of `KeepAlive`, wrapping `<Route>` from `@solidjs/router`. It automatically handles focus restoration when a user navigates back to a previously visited route.

```tsx
<KeepAliveRoute
  path="/browse"
  id="browse-screen"
  component={({ isAlive }) => <BrowseScreen isAlive={isAlive} />}
/>
```

**Focus save/restore logic**:
- `onRemove` hook: saves `activeElement()` to `savedFocusedElement`
- `onRender` hook: walks up the parent chain from `savedFocusedElement` to check if it is still a descendant of the route root (`elm`). If yes → `savedFocusedElement.setFocus()`. If no (element was destroyed or reparented) → `elm.setFocus()`.

This means a user who had focus on the 3rd item in a list, navigated to detail, and came back will return focus to that exact item — without any application code.

**`isAlive` prop injection**: the `component` prop receives `isAlive: Accessor<boolean>` alongside normal route props. Components should use this to pause background work:
```tsx
createEffect(() => {
  if (!isAlive()) return;
  // Only run when this route is visible
  startPolling();
});
```

**Preload integration**: the optional `preload` function also receives `isAlive`. On preload of an already-cached route, `setFocus()` is called on the cached node immediately.

Since [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]], `KeepAliveRoute` builds on the same Map and `createRoot` mechanism.

Focus restoration is done by calling `setFocus()` on the saved element — since [[setFocus defers focus assignment via microtask to allow children to render first]], this is safe to call immediately in `onRender` even before the route's children fully re-appear.

For an overlay/modal equivalent that stores focus without a full route transition, see [[FocusStackProvider manages focus restoration history for overlay and modal navigation patterns]] — both patterns save and restore focus but `KeepAliveRoute` is router-integrated while `FocusStackProvider` is managed manually.

Note that focus restoration via `setFocus()` also triggers `onFocusChange` in the Announcer — since [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]] diffs focus paths, returning to a preserved route may re-announce elements if `prevFocusPath` was cleared during navigation.

---

Source: [[primitives-KeepAlive]]
Domains:
- [[routing guide]]
- [[focus guide]]
