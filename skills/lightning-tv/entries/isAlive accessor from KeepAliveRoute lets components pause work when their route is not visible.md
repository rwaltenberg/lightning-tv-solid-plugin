---
description: KeepAliveRoute passes isAlive: Accessor<boolean> into component and preload props so background effects can be gated on route visibility
type: pattern
module: primitives
created: 2026-04-07
---

# isAlive accessor from KeepAliveRoute lets components pause work when their route is not visible

Components mounted via `KeepAliveRoute` receive an `isAlive` accessor as a prop. Because [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]], the reactive scope continues running even when the route is not displayed. `isAlive` gives components a way to pause expensive work while hidden.

```tsx
<KeepAliveRoute
  path="/browse"
  component={({ isAlive }) => <BrowseScreen isAlive={isAlive} />}
/>

function BrowseScreen({ isAlive }: { isAlive: Accessor<boolean> }) {
  // Pause polling when route is hidden
  createEffect(() => {
    if (!isAlive()) return;
    const interval = setInterval(fetchLatest, 5000);
    onCleanup(() => clearInterval(interval));
  });
}
```

The `isAlive` signal is set to `false` in the `onRemove` hook (when the route leaves) and back to `true` in `onRender` (when it returns). The module-level `keepAliveRouteElements` Map stores the signal so it persists across unmount/remount cycles — the same signal object is reused.

**Without `isAlive`**: animations, timers, and data fetching effects continue running in the background, wasting resources and potentially causing visual artifacts when the route becomes visible again.

---

Source: [[primitives-KeepAlive]]
Domains:
- [[routing guide]]
- [[performance guide]]
