---
description: Grid renders items using Solid's Index component rather than For, so keying is by array position — reordering items triggers prop updates on existing elements, not remounts
type: gotcha
module: primitives
created: 2026-04-07
---

# Grid uses Index not For so items are keyed by position and reordering updates in place

Grid's render implementation uses Solid's `Index` component to render the `items` array:

```tsx
<Index each={props.items}>
  {(item, index) => (
    <props.children
      item={item()}   // item is a signal — reactive to array changes at this position
      index={index}   // static integer position
      width={itemWidth()}
      height={itemHeight()}
      x={(index % columns()) * totalWidth()}
      y={Math.floor(index / columns()) * totalHeight()}
    />
  )}
</Index>
```

In Solid.js, `Index` keys by array index position:
- If the array changes at position 3, the component at position 3 receives new `item()` value
- No new component is created — the existing component updates
- `item` inside the render function is a `Signal` (accessor), not a static value

This contrasts with `For`, which keys by item identity:
- `For` would unmount and remount if items are reordered
- `Index` just updates the `item()` signal for positions that changed

**Implications:**
- Component state (animation, local signals) is NOT reset when items change
- If your item render function has expensive setup (e.g., image loading), `Index` reuses the node and just updates the data — potentially more efficient for frequently-updated lists
- If you replace the entire `items` array, all positions update their `item()` signal simultaneously

Since [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]], focus is tracked by `focusedIndex` (a position-based signal), which aligns with `Index`'s position-based keying. Focus state is preserved correctly even as item data changes.

---

Source: [[primitives-Grid]]
Domains:
- [[navigation guide]]
- [[components guide]]
