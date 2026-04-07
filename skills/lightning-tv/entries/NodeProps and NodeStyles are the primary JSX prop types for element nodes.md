---
description: NodeProps is the full prop interface for <View> elements — merges RendererNode, focus callbacks, key handlers, and ElementNode props; NodeStyles adds dollar-prefix state variants
type: api
module: core
created: 2026-04-07
---

# NodeProps and NodeStyles are the primary JSX prop types for element nodes

`NodeProps` is the TypeScript interface every `<View>` (or element node) in JSX uses. It is the union of renderer properties, focus callbacks, key event handlers, and cleaned ElementNode properties.

## NodeProps composition

```typescript
interface NodeProps
  extends RendererNode,              // renderer-level properties
    EventHandlers<DefaultKeyMap>,    // onLeft, onRight, onUp, onDown, onEnter, onLast + Release/Capture variants
    EventHandlers<KeyHoldMap>,       // onEnterHold + Release/Capture variants
    FocusNode,                       // onFocus, onBlur, onFocusChanged, onKeyPress, onKeyHold
    Partial<CleanElementNode> {      // all public ElementNode properties
  states?: NodeStates;
  style?: NodeStyles;
  children?: JSXElement | undefined;
}
```

## What CleanElementNode removes

Internal ElementNode members excluded from NodeProps:
- Private properties (all `_`-prefixed): `_type`, `_calcHeight`, `_containsFlexGrow`, etc.
- DOM-management methods: `insertChild`, `removeChild`, `destroy`, `hasChildren`, etc.
- Render-time properties: `rendered`, `renderer`, `lng`, `emit`
- Style/state runtime: `states`, `style`, `updateLayout`, `requiresLayout`
- Pre-flex state: `preFlexwidth`, `preFlexheight`

## NodeStyles — the style object type

```typescript
interface NodeStyles extends NewOmit<NodeProps, 'style'> {
  [key: `$${string}`]: NodeProps;
}
```

All NodeProps are valid style properties. Additionally, any `$`-prefixed key maps to NodeProps — this is how state variants are declared:

```typescript
<View style={{
  color: 0x333333ff,
  $focus: { color: 0xffffffff, scale: 1.1 },
  $disabled: { alpha: 0.3 },
}} />
```

## Deprecated aliases

`IntrinsicNodeProps` and `IntrinsicNodeStyleProps` are type aliases for `NodeProps` and `NodeStyles` respectively, kept for backward compatibility.

---

Related Entries:
- [[TextProps and TextStyles are the JSX prop types for text nodes]] — parallel type system for <Text> elements
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — the $focus, $hover pattern uses this index signature
- [[EventHandlers type generates key handler props from key map interfaces]] — explains onLeft, onRight, etc.
- [[FocusNode interface defines the focus lifecycle callbacks]] — onFocus, onBlur, onFocusChanged

Source: [[core-intrinsicTypes]]
Domains:
- [[core guide]]
