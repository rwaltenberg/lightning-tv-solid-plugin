---
description: Children with flexGrow expand proportionally into remaining space; triggers flexBoundary fixed; uses _containsFlexGrow flag to prevent infinite layout loops
type: api
module: core
created: 2026-04-07
---

# flexGrow distributes positive available space proportionally among children

When any child has `flexGrow > 0`, the layout engine distributes the remaining space in the container after placing all children at their natural sizes.

## Algorithm

```
availableSpace = containerSize - sumOfChildSizes - totalGapSpace
shareOfSpace[i] = (flexGrow[i] / totalFlexGrow) * availableSpace
newSize[i] = naturalSize[i] + shareOfSpace[i]
```

Children without `flexGrow` (or with `flexGrow = 0`) keep their natural size.

## Side effects

- **Auto-fixes flexBoundary**: `node.flexBoundary = node.flexBoundary || 'fixed'` — prevents the container from shrinking to children after flexGrow expands them
- **No-op when no positive space**: If `availableSpace <= 0`, flexGrow has no effect and a console warning is logged
- **Requires multiple children**: Only runs when `numProcessedChildren > 1`

## Infinite loop prevention

The `_containsFlexGrow` flag on the container node tracks state:
- First pass: sets `_containsFlexGrow = true` (allows growth)
- Next pass: sets `_containsFlexGrow = null` (prevents re-growth)
- This toggles each layout cycle to prevent unbounded expansion

## Example

```typescript
// Second child fills all remaining space
<View display="flex" flexDirection="row" width={500}>
  <View width={100} />
  <View flexGrow={1} /> {/* width becomes 400 */}
</View>

// Children share remaining space 1:2
<View display="flex" flexDirection="row" width={600}>
  <View width={100} />
  <View width={100} flexGrow={1} /> {/* gets 1/3 of 400 = ~133 extra */}
  <View width={100} flexGrow={2} /> {/* gets 2/3 of 400 = ~267 extra */}
</View>
```

---

Related Entries:
- [[flex container resizes to content by default unless flexBoundary is fixed]] — flexGrow auto-sets flexBoundary fixed
- [[flexShrink distributes overflow reduction proportionally among children]] — the counterpart for overflow situations

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
