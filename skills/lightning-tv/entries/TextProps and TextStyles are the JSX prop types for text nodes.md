---
description: TextProps is the prop interface for <Text> elements; it deliberately excludes layout props (flexDirection, gap, alignItems), focus-routing props, and visual props not applicable to text
type: api
module: core
created: 2026-04-07
---

# TextProps and TextStyles are the JSX prop types for text nodes

`TextProps` is the TypeScript interface for `<Text>` elements in JSX. It is derived from `RendererText` merged with cleaned `ElementNode` props, but with many non-applicable props explicitly omitted.

## What TextProps excludes

Compared to NodeProps, TextProps removes:
- **Layout**: `alignItems`, `direction`, `display`, `flexBoundary`, `flexDirection`, `gap`, `justifyContent`
- **Focus routing**: `forwardFocus`, `forwardStates`
- **Visual**: `linearGradient`, `src`, `scale`, `texture`, `textureOptions`
- **Children**: `children` type is `string | string[] | undefined` (not JSXElement)

## What TextProps adds

- `fontWeight?: number | string` — not on the base ElementNode; text nodes accept font weight as a string or number
- `states?: NodeStates` — state application works for text
- `style?: TextStyles` — text-specific style object
- `children?: string | string[] | undefined` — children must be text strings

## TextStyles

```typescript
interface TextStyles extends NewOmit<TextProps, 'style'> {
  [key: `$${string}`]: TextProps;
}
```

Same dollar-prefix state variant pattern as NodeStyles, but using TextProps as the state value type.

## RendererText base

`RendererText = AddColorString<Partial<Omit<lngr.ITextNodeProps, 'debug' | 'shader' | 'parent'>>>` — all Lightning renderer text properties with string color support.

```typescript
<Text
  fontSize={24}
  fontFamily="sans-serif"
  color={0xffffffff}
  style={{
    fontSize: 24,
    $focus: { fontSize: 28, color: 0xffff00ff },
  }}
>
  Hello
</Text>
```

---

Related Entries:
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — the parallel type for element nodes
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — same $-prefix pattern applies here via TextStyles

Source: [[core-intrinsicTypes]]
Domains:
- [[core guide]]
