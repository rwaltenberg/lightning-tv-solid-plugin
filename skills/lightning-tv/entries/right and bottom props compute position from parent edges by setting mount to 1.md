---
description: Setting right or bottom on an ElementNode computes x or y from the parent edge and sets mountX or mountY to 1; computed during render() not reactively
type: api
module: core
created: 2026-04-07
---

# right and bottom props compute position from parent edges by setting mount to 1

The `right` and `bottom` properties provide edge-relative positioning. They are resolved during `render()` by computing `x` or `y` from the parent's dimensions:

```typescript
if (this.right || this.right === 0) {
  props.x = parentWidth - this.right;
  props.mountX = 1;
}

if (this.bottom || this.bottom === 0) {
  props.y = parentHeight - this.bottom;
  props.mountY = 1;
}
```

Setting `right = 0` positions the element's right edge flush with the parent's right edge. Setting `right = 20` leaves 20 pixels of space from the right edge.

`mountX = 1` means the node's right edge is its anchor point, so `x = parentWidth - right` places the right edge at the correct position.

**Gotcha:** These calculations happen once during `render()`. If `parentWidth` or `parentHeight` changes after render (e.g., due to flex layout), `right` and `bottom` positioning is NOT automatically recalculated. For dynamic layouts, use flex instead.

Similarly, `center`, `centerX`, and `centerY` are computed at render time:

```typescript
if (this.center) {
  this.centerX = this.centerY = true;
}
if (this.centerX) {
  props.x += parentWidth / 2;
  props.mountX = 0.5;
}
if (this.centerY) {
  props.y += parentHeight / 2;
  props.mountY = 0.5;
}
```

`center = true` is shorthand for both `centerX` and `centerY`. The `mount` of 0.5 means the element's center is the anchor.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
