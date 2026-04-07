---
description: alignItems sets default cross-axis alignment for all children; alignSelf overrides per-child; both support flexStart, center, flexEnd; requires containerCrossSize
type: api
module: core
created: 2026-04-07
---

# alignItems and alignSelf position children on the cross axis

Cross-axis alignment (y-axis for row containers, x-axis for column containers) is controlled by `alignItems` on the container and optionally overridden per-child with `alignSelf`.

## Values

| Value | Position |
|-------|----------|
| `'flexStart'` | Aligns to start of cross axis (plus margin) |
| `'center'` | Centers within cross axis accounting for child size |
| `'flexEnd'` | Aligns to end of cross axis accounting for child margin |

## Precedence

`alignSelf` on a child overrides `alignItems` from the parent. If neither is set, no cross-axis positioning occurs (child stays at its natural position).

## Requirement: containerCrossSize must be known

Cross-axis alignment is a no-op when `containerCrossSize === 0`. If the container has no explicit height (for a row), alignment won't run. Use `_calcHeight` flag or set an explicit height.

## alignItems default with flexWrap

When `flexWrap` is set, `alignItems` defaults to `'flexStart'` if not explicitly provided:
```typescript
const align = node.alignItems || (node.flexWrap ? 'flexStart' : undefined);
```

## Example

```typescript
// All children centered vertically within a 200px-tall row
<View display="flex" flexDirection="row" height={200} alignItems="center">
  <View height={50} />   {/* y = 75 */}
  <View height={100} alignSelf="flexEnd" />  {/* y = 100 (overrides) */}
</View>
```

---

Related Entries:
- [[flexDirection controls main axis orientation and supports row-reverse and column-reverse]] — defines which axis is cross-axis
- [[flex layout engine positions children using justifyContent along the main axis]] — justifyContent handles the other axis

Source: [[core-flex]]
Domains:
- [[styling guide]]
