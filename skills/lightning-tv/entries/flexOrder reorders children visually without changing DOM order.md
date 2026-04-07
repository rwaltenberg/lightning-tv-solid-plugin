---
description: Setting flexOrder on children sorts them by ascending order value before layout runs; children without flexOrder default to 0 and maintain their DOM order within that group
type: api
module: core
created: 2026-04-07
---

# flexOrder reorders children visually without changing DOM order

`flexOrder` on a child node changes its visual position in the flex layout without altering the underlying node tree. Lower values appear first.

## Behavior

When any child has `flexOrder !== undefined`, the engine sorts the `processableChildrenIndices` array by `(a.flexOrder || 0) - (b.flexOrder || 0)` before layout.

Children without `flexOrder` default to `0`. Among same-order children, DOM order is preserved.

## Interaction with reverse

`flexOrder` sorting happens BEFORE the direction reversal step. So `row-reverse` or `direction: 'rtl'` will reverse an already-sorted order.

```typescript
<View display="flex" flexDirection="row">
  <View flexOrder={2} />  {/* appears third */}
  <View flexOrder={0} />  {/* appears first */}
  <View flexOrder={1} />  {/* appears second */}
</View>
```

## Performance note

If no child has `flexOrder`, the sort is skipped entirely. Setting `flexOrder` on any child triggers sorting for all children.

---

Related Entries:
- [[flexDirection controls main axis orientation and supports row-reverse and column-reverse]] — reversal applies after flexOrder sorting
- [[flex layout engine positions children using justifyContent along the main axis]] — order affects position in the main axis layout pass

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
