---
description: ElementNode.width returns maxWidth || w and height returns maxHeight || h; this means reading width gives the max constraint not the actual rendered size when limits are configured
type: gotcha
module: core
created: 2026-04-07
---

# width and height getters return max constraint not actual dimension when maxWidth or maxHeight is set

The `width` and `height` getters on `ElementNode` have a non-obvious behavior:

```typescript
get height(): number {
  return this.maxHeight || this.h;
}

get width(): number {
  return this.maxWidth || this.w;
}
```

When `maxWidth` is set (non-zero), `node.width` returns `maxWidth`, not `w`. When `maxHeight` is set, `node.height` returns `maxHeight`, not `h`.

**Why this matters:**

The layout system uses `node.width` as the effective width for calculating available space. If a parent has `maxWidth = 1920`, the layout engine sees a width of 1920 for space allocation, even if the actual rendered `w` is different.

This is most relevant for flex layout calculations — `calculateFlex` may read `.width` internally to determine container bounds.

**The setters:** Both setters delegate to `w` and `h` directly:

```typescript
set height(h: number) { this.h = h; }
set width(w: number) { this.w = w; }
```

So `node.width = 100` sets `w = 100`. Reading `node.width` with `maxWidth = 200` returns `200`, not `100`.

**Practical implication:** When debugging layout issues, check both `node.w` and `node.maxWidth`/`node.maxHeight` separately. Don't rely on `node.width === node.w`.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
