---
description: The roundedWithBorderWithShadow shader is cast to ShaderRoundedWithBorderAndShadowProps but the underlying lngr_shaders implementation does not handle border-gap — documented as a TODO in source
type: gotcha
module: core
created: 2026-04-07
---

# RoundedWithBorderAndShadow shader does not support border-gap property

The source code has an explicit TODO comment revealing this limitation:

```ts
// TODO: lngr_shaders.RoundedWithBorderAndShadow doesn't support border-gap
export const defaultShaderRoundedWithBorderAndShadow =
  lngr_shaders.RoundedWithBorderAndShadow as ShaderRoundedWithBorderAndShadow;
```

The TypeScript type `ShaderRoundedWithBorderAndShadowProps` includes `border-gap` (from `ShaderBorderPrefixedProps`), but the underlying `@lightningjs/renderer` shader implementation ignores it. The type cast (`as ShaderRoundedWithBorderAndShadow`) suppresses the type error without fixing the behavior.

**Consequence:** Setting `border-gap` on the `roundedWithBorderWithShadow` shader will have no visual effect. There is no runtime error or warning.

**Workaround:** Use the `roundedWithBorder` shader (without shadow) if `border-gap` is needed. If both shadow and gap are required, they must be composited via two separate elements.

Since [[ShaderBorderProps extends Lightning BorderProps with gap and inset fields]], `border-gap` is part of the officially typed API — this gotcha is not obvious from the type definitions alone.

Since [[available WebGL shader types and their registration keys]], the `roundedWithBorderWithShadow` key is the only shader with this limitation.

---

Source: [[core-shaders]]
Domains:
- [[styling guide]]
