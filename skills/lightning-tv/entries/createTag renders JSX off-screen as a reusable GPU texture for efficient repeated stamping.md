---
description: Mounts children at rootNode bottom-right corner with preventCleanup and RTT to produce a reusable texture; returns a lightweight component that stamps the texture anywhere
type: pattern
module: primitives
created: 2026-04-07
---

# createTag renders JSX off-screen as a reusable GPU texture for efficient repeated stamping

`createTag` is a performance optimization for elements that appear many times in a list (badges, labels, icons). It renders JSX once into a GPU texture, then provides a cheap component that stamps that texture at any position.

```tsx
// Create once (outside component, or in a top-level effect)
const PremiumBadge = createTag(
  <view width={60} height={24} color={0xffd700ff}>
    <text fontSize={14}>PRE</text>
  </view>
);

// Stamp anywhere, as many times as needed
function ListItem() {
  return (
    <view>
      <PremiumBadge x={10} y={10} />
    </view>
  );
}

// Clean up when no longer needed
onCleanup(() => PremiumBadge.destroy());
```

**How it works**:
1. An off-screen `<view>` is inserted at `x: rootNode.w - 1, y: rootNode.h - 1` as a direct child of `rootNode`
2. `textureOptions: { preventCleanup: true }` keeps the texture in GPU memory even when off-screen
3. `onLayout` fires after flex measurement; when `preFlexwidth !== width` (layout has settled), sets `rtt = true` and after 1ms captures `n.texture`
4. `TagComponent` renders a minimal `<view color=0xffffffff autosize texture={texture()} />` — just referencing the shared texture

**Gotcha: 1ms timeout after RTT**: `setTimeout(() => setTexture(n.texture), 1)` is required because setting `rtt = true` is asynchronous — the texture object isn't ready in the same tick.

**Gotcha: call `destroy()`**: the off-screen node stays in the tree until `.destroy()` is called. Memory leak risk if tags are created dynamically without cleanup.

Since [[VirtualRow and VirtualColumn render a sliding window over large item arrays]], `createTag` pairs naturally with Virtual components: badges that appear on every card in a large list benefit from GPU texture sharing — one upload, many references.

Since [[createSpriteMap slices a sprite sheet into named Lightning SubTexture instances]], both primitives solve the GPU texture sharing problem — createTag for JSX-defined elements, createSpriteMap for image atlases; choose based on whether the repeated element is defined in markup or as a pre-built image file.

---

Source: [[primitives-createTag]]
Domains:
- [[components guide]]
- [[performance guide]]
