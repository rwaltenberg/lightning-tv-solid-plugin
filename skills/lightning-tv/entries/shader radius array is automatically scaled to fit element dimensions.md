---
description: The internal calcFactoredRadiusArray function proportionally reduces Vec4 corner radii when they would exceed element width or height, preventing visual artifacts from oversized radii
type: architecture
module: core
created: 2026-04-07
---

# shader radius array is automatically scaled to fit element dimensions

The internal `calcFactoredRadiusArray` function in `shaders.ts` ensures that corner radius values never exceed what the element's dimensions can accommodate. This runs transparently as part of shader prop processing.

The algorithm computes a `factor` (clamped to max 1.0) and scales all four corners proportionally:

```ts
let factor = Math.min(
  width / Math.max(width, radius[0] + radius[1]),   // top-left + top-right <= width
  width / Math.max(width, radius[2] + radius[3]),   // bottom-left + bottom-right <= width
  height / Math.max(height, radius[0] + radius[3]), // top-left + bottom-left <= height
  height / Math.max(height, radius[1] + radius[2]), // top-right + bottom-right <= height
  1,
);
```

Each pair of adjacent corners is constrained in the relevant axis. The proportional scaling preserves the relative ratio between corners (e.g., if top-left is twice top-right, it stays twice).

**Implication:** Setting a `radius` of `[100, 100, 100, 100]` on a 60x60 element does not error — it silently scales down to approximately `[30, 30, 30, 30]`. The element will look correct but the radius values in your style won't be applied literally.

**The `toValidVec4` helper** normalizes shorthand radius values before this scaling:
- Single number → `[n, n, n, n]`
- 2-element array → `[a, b, a, b]`
- 3-element array → `[a, b, c, a]`
- 4-element array → used as-is

Since [[available WebGL shader types and their registration keys]] includes the rounded shaders that use radius values, this scaling applies to all rounded corner configurations.

---

Source: [[core-shaders]]
Domains:
- [[styling guide]]
