---
description: If a flex container contains an ElementText node with text content but no width or height set, the flex engine aborts layout entirely and returns false
type: gotcha
module: core
created: 2026-04-07
---

# flex engine returns false early when ElementText child has text but no explicit dimensions

The flex layout engine has a specific early-exit condition for text nodes with unset dimensions. When an `ElementText` node (type `'textNode'`) has a `text` value but neither `width` nor `height` is set, the entire container layout is aborted.

## The check

```typescript
if (isElementText(c) && c.text && !(c.width || c.height)) {
  return false;
}
```

## Why this exists

Text nodes need their size measured before layout can run. Without explicit dimensions, the text node's size is unknown at layout time — it depends on font metrics that may be resolved asynchronously. Returning `false` defers layout until the text node has been measured and its dimensions populated.

## Practical implication

A flex container that mixes `<Text>` children without explicit `width`/`height` alongside positioned siblings may silently fail to lay out. The container returns `false` (no layout) rather than throwing an error.

## Workaround

Set explicit `width` or `height` on text nodes in flex containers, or use a wrapper element:

```typescript
// Problem: layout aborts
<View display="flex" flexDirection="row">
  <Text>Label</Text>  {/* no width/height — aborts layout */}
  <View width={200} />
</View>

// Fix: wrap in sized container
<View display="flex" flexDirection="row">
  <View width={100}><Text>Label</Text></View>
  <View width={200} />
</View>
```

---

Related Entries:
- [[node type discriminators allow the flex engine to skip text nodes]] — isElementText is used in this check
- [[flexItem false excludes a child from flex layout calculations]] — text nodes are also excluded from flex item processing (different path)

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
