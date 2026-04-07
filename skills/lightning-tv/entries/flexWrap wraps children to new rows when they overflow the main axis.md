---
description: flexWrap='wrap' starts new rows/columns when overflow is detected; wrap-reverse fills from the end; both resize the container cross-dimension to fit all rows
type: api
module: core
created: 2026-04-07
---

# flexWrap wraps children to new rows when they overflow the main axis

When `flexWrap='wrap'` is set, children that would overflow the main axis start on a new row (or column for column-direction). The container's cross-dimension grows to accommodate all wrapped rows.

## Values

- `'wrap'` — wraps from start; new rows appear below (row) or to the right (column)
- `'wrap-reverse'` — wraps from end; children start at the cross-axis end and new rows appear toward the start

## Overflow detection

A new row starts when:
```
currentPos + childTotalMainSize > containerSize && currentPos > padding
```
The second condition prevents wrapping when the container is smaller than a single child.

## Cross-dimension resize

After wrapping, the engine calculates `finalCrossSize` and updates the container's cross-dimension:
- Stores old value in `preFlex{crossDimension}` before resizing
- Sets `containerUpdated = true`

## rowGap / columnGap

The gap between wrapped rows uses `columnGap` (for row-direction) or `rowGap` (for column-direction), falling back to `gap` if not set separately.

## Interaction with container auto-resize

Container auto-resize (`flexBoundary !== 'fixed'`) is disabled when `flexWrap === 'wrap'`. The main axis size stays fixed; only the cross axis is allowed to expand.

## alignItems default

When `flexWrap` is set, `alignItems` defaults to `'flexStart'` if not explicitly configured.

```typescript
<View display="flex" flexDirection="row" width={300} flexWrap="wrap">
  <View width={200} height={50} /> {/* row 1 */}
  <View width={200} height={50} /> {/* row 2 — overflows, wraps */}
</View>
{/* container height becomes 100 */}
```

---

Related Entries:
- [[flex container resizes to content by default unless flexBoundary is fixed]] — wrap disables main-axis auto-resize
- [[alignItems and alignSelf position children on the cross axis]] — wrap sets alignItems default
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — VirtualGrid uses flexWrap:'wrap' as its core 2D layout mechanism; items flow into rows naturally based on their width

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
