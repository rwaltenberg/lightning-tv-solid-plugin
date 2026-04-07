---
description: Wraps children with the preserve prop and stores them in a module-level Map keyed by id, returning the same children on re-mount
type: architecture
module: primitives
created: 2026-04-07
---

# KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation

`KeepAlive` keeps both the SolidJS reactive scope and the Lightning renderer node tree alive when a component is navigated away from. Without it, route changes trigger reactive disposal, freeing GPU memory and resetting all state.

```tsx
<KeepAlive id="home-screen">
  <HomeScreen />
</KeepAlive>
```

**Props**:
```ts
interface KeepAliveProps {
  id: string;                          // unique key for this preserved element
  shouldDispose?: (key: string) => boolean;  // force teardown if true
  onRemove?: ElementNode['onRemove'];   // called when route leaves (default: alpha=0)
  onRender?: ElementNode['onRender'];   // called when route returns (default: alpha=1)
  transition?: ElementNode['transition']; // default: { alpha: true }
}
```

**How state is preserved**:
1. First mount: `createRoot` creates a reactive scope with a `dispose` function. Children are wrapped in a `<view preserve>` which prevents the Lightning node from being destroyed on removal.
2. The `KeepAliveElement` (containing `owner`, `children`, `dispose`, `isAlive` signal) is stored in a module-level `Map<string, KeepAliveElement>`.
3. Subsequent mounts: the stored `children` are returned directly — no re-render, no state reset.

**Visibility control**: `onRemove` sets `alpha = 0` (invisible but still in tree); `onRender` sets `alpha = 1`. The `transition: { alpha: true }` provides a fade effect. `setIsAlive` is called in both hooks.

Because [[ElementNode render method controls the two-phase lifecycle from accumulation to live rendering]], the `preserve` prop is what prevents the Lightning node from being freed during the removal phase. Specifically, since [[preserve property prevents node destruction by holding the delete counter at zero]], setting `preserve` on the route's root view pins `_queueDelete` to 0 so `flushDeleteQueue` never calls `destroy()` during navigation.

Note that even though nodes are preserved, [[onDestroy can return a promise to delay node destruction for exit animations]] is NOT called for preserved nodes — `destroy()` is never invoked, so exit animation promises would need to be handled via the `onRemove` prop instead.

---

Source: [[primitives-KeepAlive]]
Domains:
- [[routing guide]]
- [[components guide]]
