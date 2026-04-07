---
description: Eight built-in WebGL shaders available via registerDefaultShaders — rounded, shadow, roundedWithBorder, roundedWithShadow, roundedWithBorderWithShadow, holePunch, radialGradient, linearGradient
type: api
module: core
created: 2026-04-07
---

# available WebGL shader types and their registration keys

Lightning TV/Solid ships eight default WebGL shaders, each registered under a string key. They are registered by calling `registerDefaultShaders(shManager)` or their individual `registerDefaultShader*` functions.

| Registration Key | Default Instance | Props Type |
|---|---|---|
| `'rounded'` | `defaultShaderRounded` | `ShaderRoundedProps` |
| `'shadow'` | `defaultShaderShadow` | `ShaderShadowProps` |
| `'roundedWithBorder'` | `defaultShaderRoundedWithBorder` | `ShaderRoundedWithBorderProps` |
| `'roundedWithShadow'` | `defaultShaderRoundedWithShadow` | `ShaderRoundedWithShadowProps` |
| `'roundedWithBorderWithShadow'` | `defaultShaderRoundedWithBorderAndShadow` | `ShaderRoundedWithBorderAndShadowProps` |
| `'holePunch'` | `defaultShaderHolePunch` | `ShaderHolePunchProps` |
| `'radialGradient'` | `defaultShaderRadialGradient` | `ShaderRadialGradientProps` |
| `'linearGradient'` | `defaultShaderLinearGradient` | `ShaderLinearGradientProps` |

The composite shader prop types use TypeScript prefixed mapped types:
- Shadow props appear as `shadow-color`, `shadow-offsetX`, etc.
- Border props appear as `border-color`, `border-width`, `border-gap`, `border-inset`, etc.

Since [[ShaderBorderProps extends Lightning BorderProps with gap and inset fields]], the border shader accepts two extra properties not in the base renderer API.

Since [[RoundedWithBorderAndShadow shader does not support border-gap property]], the `roundedWithBorderWithShadow` key is the only shader with a known prop limitation.

Since [[shaders are silently skipped when DOM renderer is active]], none of these register in DOM mode.

---

Source: [[core-shaders]]
Domains:
- [[styling guide]]
