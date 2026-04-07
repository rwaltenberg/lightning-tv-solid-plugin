---
description: Any $-prefixed key in a style object overrides properties when the matching state is active; state names are arbitrary strings matching the NodeStates set on the node
type: pattern
module: core
created: 2026-04-07
---

# dollar-prefix state keys in NodeStyles apply style variants based on active states

Style objects in Lightning TV/Solid use `$`-prefixed keys to declare state-conditional overrides. When a node has an active state matching the key (minus the `$`), the override properties are applied.

## Type system

```typescript
interface NodeStyles extends NewOmit<NodeProps, 'style'> {
  [key: `$${string}`]: NodeProps;  // any $-prefixed key maps to NodeProps
}
```

The TypeScript index signature `[key: \`$${string}\`]: NodeProps` makes any `$`-prefixed key valid in a style object.

## Common state keys

| Key | Active when |
|-----|-------------|
| `$focus` | Node or ancestor has focus |
| `$hover` | Mouse hover (non-TV usage) |
| `$active` | Node is in active/pressed state |
| `$disabled` | Node has disabled state |

State names are arbitrary — they match strings in the node's `NodeStates` set.

## Example

```typescript
const buttonStyle = {
  width: 200,
  height: 60,
  color: 0x333333ff,
  borderRadius: 8,
  $focus: {
    color: 0x0066ffff,
    scale: 1.05,
  },
  $disabled: {
    alpha: 0.4,
    color: 0x666666ff,
  },
};

<View style={buttonStyle} states={['focus']} />
// Applies $focus overrides because 'focus' is in states
```

## DollarString type

`type DollarString = \`$${string}\`` is exported from intrinsicTypes for use in style system utilities.

---

Related Entries:
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — NodeStyles contains this index signature
- [[TextProps and TextStyles are the JSX prop types for text nodes]] — TextStyles uses the same pattern
- [[forwardStates propagates parent state array to all children on every state change]] — enables child elements to apply $focus styles without receiving focus directly
- [[borderBox directive injects a focus ring child into Lightning nodes using onFocusChanged]] — an alternative focus feedback mechanism that creates a child node instead of applying $focus style overrides
- [[States class extends Array to manage dollar-prefixed state strings]] — the runtime object that holds active state strings and triggers style variant application
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — Row items typically use $focus style variants to visually highlight the selected item
- [[Marquee component scrolls overflowing text using two synchronized looping animations]] — Marquee's `marquee` prop is commonly driven by a $focus-derived signal

Source: [[core-intrinsicTypes]]
Domains:
- [[core guide]], [[styling guide]]
