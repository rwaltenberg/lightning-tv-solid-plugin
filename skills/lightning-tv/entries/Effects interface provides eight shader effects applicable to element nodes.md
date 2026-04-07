---
description: The Effects interface lists all composable visual shader effects — linearGradient, radialGradient, holePunch, shadow, rounded, borderRadius, border, and per-side borders
type: api
module: core
created: 2026-04-07
---

# Effects interface provides eight shader effects applicable to element nodes

The `Effects` interface (aliased as `StyleEffects`) enumerates all visual shader effects that can be applied to element nodes. Each is optional and uses partial props from the corresponding shader.

## Available effects

| Effect key | Description |
|------------|-------------|
| `linearGradient` | Linear gradient fill using ShaderLinearGradientProps |
| `radialGradient` | Radial gradient fill using ShaderRadialGradientProps |
| `holePunch` | Punches a transparent hole through the element (for overlay cutouts) |
| `shadow` | Drop shadow using ShaderShadowProps |
| `rounded` | Rounded corners via shader |
| `borderRadius` | Shorthand for rounded corners, accepts `number | number[]` |
| `border` | Full-perimeter border using ShaderBorderProps |
| `borderTop`, `borderBottom`, `borderLeft`, `borderRight` | Per-side borders |

## BorderStyleObject

Border effects accept a rich configuration object:
```typescript
interface BorderStyleObject {
  width?: number | [number, number, number, number];  // per-side widths
  gap?: number;      // gap between border and element edge
  align?: 'inside' | 'outside' | 'center';  // border placement
  fill?: number | string;  // color fill
  // ...plus all lngr.BorderProps with string color support
}
```

## SingleBorderStyleObject

For per-side borders (`borderTop`, etc.):
```typescript
interface SingleBorderStyleObject {
  width?: number;   // single number only
  w?: number;       // shorthand
  gap?: number;
  align?: 'inside' | 'outside' | 'center';
  fill?: number | string;
}
```

## AddColorString

All border and shader color props accept `string | number` via the `AddColorString<T>` utility type:
```typescript
// Both valid:
border={{ color: 0xff0000ff }}
border={{ color: '#ff0000' }}
```

## Usage

```typescript
<View
  effects={{
    shadow: { offsetX: 0, offsetY: 4, blur: 8 },
    rounded: { radius: 12 },
  }}
  border={{ color: 0xffffff33, width: 2 }}
/>
```

---

Related Entries:
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — Effects are applied through NodeProps

Source: [[core-intrinsicTypes]]
Domains:
- [[styling guide]]
