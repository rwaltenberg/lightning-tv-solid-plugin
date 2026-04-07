---
description: The effects setter compiles visual effects (rounded, border, shadow) into a shader type string like 'roundedWithBorderWithShadow'; defers shader creation until render() if not yet rendered
type: architecture
module: core
created: 2026-04-07
---

# effects setter builds shader type by composing feature flags and converts before render

The `effects` property maps visual effect descriptors to Lightning shader programs. The `convertToShader` function builds the shader type name by concatenating feature flags:

```typescript
export function convertToShader(_node: ElementNode, v: StyleEffects): IRendererShader {
  let type = 'rounded';
  if (v.border) type += 'WithBorder';
  if (v.shadow) type += 'WithShadow';
  return renderer.createShader(type, v);
}
```

Possible shader types: `'rounded'`, `'roundedWithBorder'`, `'roundedWithShadow'`, `'roundedWithBorderWithShadow'`.

The effects setter:

```typescript
set effects(v: StyleEffects) {
  if (!SHADERS_ENABLED) return;
  let target = this.lng.shader || {};
  // ... if already has shader props, use those
  if (v.rounded) target.radius = v.rounded.radius;
  if (v.borderRadius) target.radius = v.borderRadius;
  if (v.border) parseAndAssignShaderProps('border', v.border, target);
  // ... other effects
  if (this.rendered) {
    if (!this.lng.shader) {
      this.lng.shader = Config.convertToShader(this, target);  // uses config override
    }
    // else update in place (DOM renderer forces re-trigger via setter)
  } else {
    this.lng.shader = target;  // accumulate; converted at render time
  }
}
```

**Two-phase behavior:**
- Before render: stores raw props in `lng.shader` as a plain object
- After render: creates the actual shader via `Config.convertToShader` (which defaults to `convertToShader` but can be overridden)

**SHADERS_ENABLED guard:** The entire effects system is guarded by this flag. When disabled (e.g., in certain DOM renderer modes), all effects are silently ignored.

The `shaderAccessor` function (used for `border`, `shadow`, `rounded`, `borderRadius`, etc.) also supports transitions — setting `border` can animate if `transition.borderRadius` or `transition.border` is configured.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
