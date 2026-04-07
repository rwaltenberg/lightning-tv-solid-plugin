---
description: Four flexDirection values map to two axes; reverse variants reverse child order visually without reordering the DOM; RTL direction also reverses layout
type: api
module: core
created: 2026-04-07
---

# flexDirection controls main axis orientation and supports row-reverse and column-reverse

`flexDirection` determines which axis children stack along and whether they are visually reversed.

## Values

| Value | Main axis | Cross axis | Reversed? |
|-------|-----------|------------|-----------|
| `'row'` (default) | x (left→right) | y (top→bottom) | No |
| `'row-reverse'` | x (right→left) | y | Yes |
| `'column'` | y (top→bottom) | x (left→right) | No |
| `'column-reverse'` | y (bottom→top) | x | Yes |

## Implementation

The engine maps direction to dimension strings:
- `isRow` → `dimension = 'width'`, `crossDimension = 'height'`, `prop = 'x'`, `crossProp = 'y'`
- `!isRow` → `dimension = 'height'`, `crossDimension = 'width'`, `prop = 'y'`, `crossProp = 'x'`

Reverse variants and `direction === 'rtl'` both call `processableChildrenIndices.reverse()` before layout — they produce identical layout results by flipping the processing order.

## RTL interaction

Setting `direction: 'rtl'` on a node triggers the same reversal as `row-reverse`. This enables locale-driven layout direction changes without changing `flexDirection` itself.

```typescript
<View display="flex" flexDirection="column-reverse">
  <View /> {/* appears at bottom */}
  <View /> {/* appears above */}
</View>
```

---

Related Entries:
- [[flex layout engine positions children using justifyContent along the main axis]] — justifyContent operates along the axis defined here
- [[alignItems and alignSelf position children on the cross axis]] — cross axis is the perpendicular of this axis
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — Row uses flexDirection:'row' (the default)
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — Column explicitly sets flexDirection:'column' to stack items vertically

Source: [[core-flex]]
Domains:
- [[styling guide]]
