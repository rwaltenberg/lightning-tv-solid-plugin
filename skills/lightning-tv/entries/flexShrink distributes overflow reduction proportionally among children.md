---
description: Only in flexLayout.ts — when children overflow the container, flexShrink shrinks them using a weighted ratio based on flexShrink value times child size, respecting minWidth/minHeight
type: api
module: core
created: 2026-04-07
---

# flexShrink distributes overflow reduction proportionally among children

When children collectively overflow the container (`availableSpace < 0`) and any child has `flexShrink > 0`, the engine reduces child sizes to fit. This is only available in `flexLayout.ts`, not the older `flex.ts`.

## Algorithm

The shrink uses a weighted ratio, not a simple proportion:

```
totalScaledShrinkFactor = sum(flexShrink[i] * naturalSize[i])
shrinkRatio[i] = (flexShrink[i] * naturalSize[i]) / totalScaledShrinkFactor
sizeReduction[i] = shrinkRatio[i] * Math.abs(availableSpace)
newSize[i] = max(naturalSize[i] - sizeReduction[i], minWidth/minHeight)
```

Larger children shrink more in absolute terms. This matches CSS flexbox behavior.

## Key behaviors

- **minWidth/minHeight respected**: A child will not shrink below its minimum dimension
- **Auto-fixes flexBoundary**: Same as flexGrow — sets `flexBoundary = 'fixed'`
- **Shares `_containsFlexGrow` flag**: Both grow and shrink use the same loop-prevention mechanism
- **Only when `numProcessedChildren > 1`**: Same guard as flexGrow

## Example

```typescript
// Children must fit in 300px but total natural size is 400px
<View display="flex" flexDirection="row" width={300} flexBoundary="fixed">
  <View width={200} flexShrink={1} /> {/* shrinks more (200/400 * 100 = 50px) */}
  <View width={200} flexShrink={1} /> {/* shrinks same amount */}
  {/* Both end up at 150px */}
</View>
```

---

Related Entries:
- [[flexGrow distributes positive available space proportionally among children]] — the counterpart for expansion; shares implementation state
- [[flexLayout.ts adds flexBasis flexShrink and per-side padding over the original flex implementation]] — flexShrink is exclusive to the enhanced implementation

Source: [[core-flexLayout]]
Domains:
- [[styling guide]]
