---
description: When _calcHeight is true on a row flex container and flexCrossBoundary is not set, the container height is set to the tallest child's height before alignment runs
type: architecture
module: core
created: 2026-04-07
---

# flex engine auto-calculates row container height from tallest child via _calcHeight flag

Row flex containers can automatically set their height to match the tallest child. This runs as a pre-alignment step before cross-axis positioning.

## Conditions for auto-height

All three must be true:
1. `isRow` — container is `flexDirection: 'row'` or `'row-reverse'`
2. `node._calcHeight === true` — the internal flag is set
3. `!node.flexCrossBoundary` — not explicitly locked

## Mechanism

```typescript
if (isRow && node._calcHeight && !node.flexCrossBoundary) {
  let maxHeight = 0;
  for (let idx = 0; idx < numProcessedChildren; idx++) {
    if (childCrossSizes[idx] > maxHeight) maxHeight = childCrossSizes[idx];
  }
  const newHeight = maxHeight || node.height;
  if (newHeight !== node.height) {
    containerUpdated = true;
    node.height = containerCrossSize = newHeight;
  }
}
```

The updated `containerCrossSize` is then used for `alignItems` / `alignSelf` calculations in the same layout pass.

## Distinction from flexBoundary

`flexBoundary` controls main-axis resize. `flexCrossBoundary` / `_calcHeight` control cross-axis resize. They are independent. A container can have `flexBoundary: 'fixed'` (fixed width) but still auto-calculate height.

## How _calcHeight is set

This is an internal flag (`_` prefix = private). It is set by higher-level abstractions (Row component, Column component) rather than being a user-facing prop.

---

Related Entries:
- [[flex container resizes to content by default unless flexBoundary is fixed]] — main-axis counterpart to cross-axis auto-sizing
- [[alignItems and alignSelf position children on the cross axis]] — cross-axis alignment uses the auto-calculated height

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
