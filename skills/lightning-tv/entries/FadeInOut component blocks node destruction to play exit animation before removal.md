---
description: Uses onDestroy returning a Promise and rtt=true to animate alpha to 0 before the Lightning node is freed; onCreate animates alpha from 0 to 1
type: pattern
module: primitives
created: 2026-04-07
---

# FadeInOut component blocks node destruction to play exit animation before removal

`FadeInOut` wraps a `<View>` in `<Show when keyed>` and handles alpha transitions automatically on mount and unmount.

```tsx
<FadeInOut when={isVisible()} transition={{ duration: 400, easing: 'ease-out' }}>
  <MyContent />
</FadeInOut>
```

**Props**:
```ts
{
  when?: boolean;
  transition?: { duration?: number; easing?: string };  // defaults: 250ms, ease-in-out
}
```

**Enter** (`onCreate`): immediately sets `alpha = 0`, then calls `el.animate({ alpha: 1 }, config).start()`.

**Exit** (`onDestroy`):
1. Sets `el.rtt = true` — freezes the visual content as a texture to avoid glitching during alpha animation
2. Calls `el.animate({ alpha: 0 }, config).start().waitUntilStopped()`
3. Returns the resulting `Promise<void>` — the framework waits for this promise before freeing the node

The `<Show keyed>` is essential: it ensures the child is fully destroyed and recreated (triggering lifecycle hooks) when `when` toggles, rather than just hidden.

Since [[onDestroy can return a promise to delay node destruction for exit animations]], the framework natively supports this pattern. `FadeInOut` is a convenience wrapper that implements it correctly.

The standalone helpers `fadeIn(el)` and `fadeOut(el)` are also exported for imperative use on existing `ElementNode` references.

Since [[animatable number properties route through transition system before reaching the renderer]], the alpha animation used by FadeInOut only works after the node is rendered — `onCreate` correctly fires post-render.

---

Source: [[primitives-FadeInOut]]
Domains:
- [[components guide]]
- [[styling guide]]
