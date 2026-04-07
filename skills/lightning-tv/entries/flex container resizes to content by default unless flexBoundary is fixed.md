---
description: Without flexBoundary='fixed', flexStart containers auto-resize to children; the original dimension is preserved in preFlex{dimension} for Row/Column restoration
type: architecture
module: core
created: 2026-04-07
---

# flex container resizes to content by default unless flexBoundary is fixed

When `justifyContent` is `'flexStart'` (the default), a flex container expands or contracts to exactly fit its children. This is the default behavior — you must opt out of it with `flexBoundary: 'fixed'`.

## Mechanism

1. After positioning children, the engine calculates `calculatedSize = lastPosition - gap + padding`
2. If `calculatedSize !== current dimension`, the container is resized and `true` is returned (triggering parent re-layout)
3. Before resizing, the original size is stored in `node.preFlex{dimension}` — e.g., `preFlexwidth` or `preFlexheight`

## When flexBoundary auto-activates

Using `flexGrow` on any child automatically sets `flexBoundary = 'fixed'` on the container if not already set. This prevents infinite loops where grow expands children → container resizes → grow recalculates.

## Opting out

```typescript
// Container stays at exactly 500px regardless of child count
<View display="flex" width={500} flexBoundary="fixed">
  {items.map(i => <View width={100} />)}
</View>
```

## flexWrap exception

Container auto-resize is disabled when `flexWrap === 'wrap'`. Wrapped containers manage their cross-dimension resize separately based on row/column count.

---

Related Entries:
- [[flex layout engine positions children using justifyContent along the main axis]] — flexStart is the only justify mode that triggers container resize
- [[flexGrow distributes positive available space proportionally among children]] — flexGrow auto-activates flexBoundary fixed

Source: [[core-flex]]
Domains:
- [[styling guide]]
