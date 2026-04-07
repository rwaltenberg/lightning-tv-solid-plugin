---
description: During render(), if w or h is NaN (not set), they default to parentWidth - x and parentHeight - y respectively; flexGrow items get w = 0 instead to let flex distribute space
type: gotcha
module: core
created: 2026-04-07
---

# element node NaN width and height default to parent dimensions minus position offset

When rendering an element node without a `texture`, if `w` or `h` is `NaN` (not set), the renderer uses parent dimensions as fallback:

```typescript
if (isNaN(props.w as number)) {
  props.w = node.flexGrow ? 0 : parentWidth - props.x;
  node._calcWidth = true;
}

if (isNaN(props.h as number)) {
  props.h = parentHeight - props.y;
  node._calcHeight = true;
}
```

The `_calcWidth` and `_calcHeight` flags track that these were auto-computed (used elsewhere in the system).

**Default sizing behavior:**

- An element with no explicit `w` that is at `x = 100` inside a 1920px parent gets `w = 1820`
- An element with no explicit `h` that is at `y = 50` inside a 1080px parent gets `h = 1030`
- A flex item with `flexGrow` gets `w = 0` regardless — flex distributes space to it

**Gotcha:** This auto-sizing only applies to non-texture nodes. If `props.texture` is set, no default sizing is applied (textures define their own dimensions).

**Gotcha:** `isNaN` returns `true` for `undefined`, which is the initial state when no `w` or `h` is set. Setting `w = 0` explicitly is NOT the same as not setting it — `isNaN(0) === false`, so `w = 0` stays `0`.

This behavior means containers without explicit sizes automatically fill the remaining space of their parent, similar to CSS `width: 100%` offset by the element's position.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
