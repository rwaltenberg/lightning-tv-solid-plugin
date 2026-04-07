---
description: Solid's ShaderBorderProps adds gap (distance between border and element edge) and inset (boolean, default true) on top of the base renderer BorderProps
type: api
module: core
created: 2026-04-07
---

# ShaderBorderProps extends Lightning BorderProps with gap and inset fields

The Solid framework wraps the base `@lightningjs/renderer` `BorderProps` with two additional fields:

```ts
export interface ShaderBorderProps extends lngr.BorderProps {
  /** Distance between the border and element edges. */
  gap: number;
  /**
   * If false, border is drawn outside the element.
   * If true, border is drawn inside the element.
   * @default true
   */
  inset: boolean;
}
```

When used with the `roundedWithBorder` shader, these fields control border placement:

```ts
// Border 4px inside the element edges, with a 2px gap
{
  effects: {
    roundedWithBorder: {
      radius: 12,
      'border-width': 4,
      'border-gap': 2,
      'border-inset': true,
      'border-color': 0xff0000ff,
    }
  }
}
```

**Note on prefixed prop types:** When combined into composite types, border props are prefixed with `border-`:

```ts
type ShaderBorderPrefixedProps = {
  [P in keyof ShaderBorderProps as `border-${P}`]: ShaderBorderProps[P];
};
// results in: 'border-gap', 'border-inset', 'border-width', 'border-color', etc.
```

Since [[RoundedWithBorderAndShadow shader does not support border-gap property]], using `border-gap` with the combined border+shadow shader has no effect.

Since [[available WebGL shader types and their registration keys]] includes the list of all shader keys, `roundedWithBorder` and `roundedWithBorderWithShadow` are the relevant targets for border props.

---

Source: [[core-shaders]]
Domains:
- [[styling guide]]
