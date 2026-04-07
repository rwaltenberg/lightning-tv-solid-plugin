---
description: Sets `view.preserve = true` on the ElementNode so the Lightning renderer retains the node when it would otherwise be destroyed; shows/hides via onRender and onRemove hooks
type: architecture
module: primitives
created: 2026-04-07
---

`Preserve` wraps a `<view>` and instructs the Lightning renderer not to destroy the node when it is removed from the component tree:

```tsx
<Preserve width={1920} height={1080}>
  <MyPage />
</Preserve>
```

## What preserve Does

Setting `view.preserve = true` on an ElementNode changes the renderer's removal behavior: instead of releasing GPU resources and destroying the node hierarchy, the renderer keeps the node alive. This is the foundation for instant navigation back to previously visited pages.

## Lifecycle Integration

```ts
view.onRender ??= () => { view.hidden = false; };
view.onRemove ??= () => { view.hidden = true; };
```

- `onRender`: fires when the node is re-added to the scene — unhides it
- `onRemove`: fires when the node is logically removed — hides it

The `??=` operator means these defaults only apply if the parent component didn't set its own handlers.

## onCleanup and Ultimate Destruction

```ts
s.onCleanup(() => { view.destroy(); });
```

When the Solid reactive scope that owns this `Preserve` is cleaned up, the node IS eventually destroyed. `Preserve` defers destruction but does not prevent it. The routing layer must keep the owning Solid scope alive for preservation to persist across navigation.

## Contrast with Visible

Since [[Visible component toggles ElementNode hidden instead of destroying and recreating children]], `Visible` manages show/hide within a single page. `Preserve` manages show/hide across route transitions at the renderer level. The key difference: `Visible`'s children live inside the current Solid component tree; `Preserve`'s node lives in the renderer independently of the current tree.

---

Source: [[primitives-Preserve]]
Domains:
- [[performance guide]]
- [[components guide]]
- [[routing guide]]
