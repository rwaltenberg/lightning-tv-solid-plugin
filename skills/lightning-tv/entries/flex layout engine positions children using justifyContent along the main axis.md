---
description: Six justifyContent values — flexStart, flexEnd, center, spaceBetween, spaceAround, spaceEvenly — control child distribution along the main axis
type: api
module: core
created: 2026-04-07
---

# flex layout engine positions children using justifyContent along the main axis

The flex layout engine distributes children along the main axis (x for row, y for column) according to the `justifyContent` property. Default is `'flexStart'`.

## Available values

| Value | Behavior |
|-------|----------|
| `'flexStart'` | Stacks children from the start; container resizes to content unless `flexBoundary: 'fixed'` |
| `'flexEnd'` | Places children from the end, working backward |
| `'center'` | Centers the entire group within the container |
| `'spaceBetween'` | Equal gaps between items; no outer gaps |
| `'spaceAround'` | Half-gap at edges, full gap between items |
| `'spaceEvenly'` | Equal space before every item and after the last |

## Gap interaction

All justify modes respect the `gap` property for spacing between items. `spaceBetween`, `spaceAround`, and `spaceEvenly` calculate their distribution based on `containerSize - totalItemSize - padding`.

## Container auto-resize

`flexStart` is special: when `flexBoundary !== 'fixed'`, the container shrinks or grows to exactly fit its children. It stores the original size in `preFlex{dimension}` before resizing and returns `true` to signal the parent that a re-layout pass is needed.

```typescript
// Container grows to fit content (default)
<View display="flex" flexDirection="row">
  <View width={100} />
  <View width={200} />
</View>

// Container stays fixed, children distributed
<View display="flex" flexDirection="row" width={500} justifyContent="spaceBetween">
  <View width={100} />
  <View width={100} />
</View>
```

---

Related Entries:
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — Row uses display:'flex' with justifyContent defaulting to flexStart
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — Column also uses display:'flex' with flexStart distribution along the y-axis
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — VirtualGrid uses flex layout with flexWrap to arrange items into rows

Source: [[core-flex]]
Domains:
- [[styling guide]]
