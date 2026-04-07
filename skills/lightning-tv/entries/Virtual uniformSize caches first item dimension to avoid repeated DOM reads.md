---
description: When `uniformSize: true` (the default), the first measured item width/height is cached in `cachedScaledSize` and reused for all subsequent window-shift calculations
type: pattern
module: primitives
created: 2026-04-07
---

Every time the Virtual window shifts, the component needs to know how far to move the container. This requires measuring the current item's rendered dimension (width for VirtualRow, height for VirtualColumn) plus gap. Without caching, this would trigger a layout read on every navigation event.

## The Optimization

`uniformSize` (default: `true`) enables dimension caching:

```ts
const uniformSize = createMemo(() => props.uniformSize !== false);

function computeSize(selected: number = 0) {
  if (uniformSize() && cachedScaledSize) {
    return cachedScaledSize;
  }
  // ... measure from viewRef.children[selected]
  cachedScaledSize = itemSize * (props.factorScale ? scale : 1) + gap;
  return cachedScaledSize;
}
```

After the first measurement, subsequent calls return `cachedScaledSize` immediately without touching the renderer.

## factorScale Integration

When `factorScale: true`, the cached size includes the item's focus scale:

```ts
const focusStyle = prevSelectedChild.style?.focus as lng.NodeStyles;
const scale = focusStyle?.scale ?? prevSelectedChild.scale ?? 1;
const scaledSize = itemSize * scale + gap;
```

This ensures the shift distance accounts for the visual size increase when an item is focused, preventing misalignment between the focused item's visual position and the container's shifted position.

## When to Disable

Set `uniformSize: false` when items have variable sizes (e.g., a row mixing card sizes). With `false`, the component re-measures on every window shift. This is more accurate but slightly slower.

---

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
