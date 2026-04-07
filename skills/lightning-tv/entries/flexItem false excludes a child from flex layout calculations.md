---
description: Setting flexItem=false on a child removes it from flex layout processing entirely; the child retains its own position/size but does not participate in distribution
type: api
module: core
created: 2026-04-07
---

# flexItem false excludes a child from flex layout calculations

Setting `flexItem={false}` on an element that is a child of a flex container opts that child out of the flex layout system. It is skipped during index collection and does not contribute to gap calculations, size sums, or position assignments.

## Use cases

- Absolutely positioned overlays within a flex container
- Decorative elements that should not affect sibling layout
- Fixed-position children that manage their own x/y

## Behavior

The child is filtered out during the initial pass:
```typescript
if (isTextNode(c) || c.flexItem === false) {
  continue; // skipped from processableChildrenIndices
}
```

The child's `x`, `y`, `width`, and `height` are not modified by the flex engine. Its existing values remain unchanged.

## Text nodes are also excluded

`isTextNode(c)` applies the same exclusion to internal text nodes automatically — they are never flex items.

```typescript
<View display="flex" flexDirection="row" width={400}>
  <View width={100} />   {/* flex item, x=0 */}
  <View width={100} flexItem={false} />  {/* NOT a flex item, keeps own x */}
  <View width={100} />   {/* flex item, x=100 (sibling above skipped) */}
</View>
```

---

Related Entries:
- [[flex layout engine positions children using justifyContent along the main axis]] — this exclusion affects what the engine sees
- [[node type discriminators allow the flex engine to skip text nodes]] — text nodes are excluded by the same mechanism

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
