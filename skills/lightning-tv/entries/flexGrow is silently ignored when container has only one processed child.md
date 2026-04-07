---
description: The flex engine gates the flexGrow distribution block on numProcessedChildren > 1, so a single-child flex container with flexGrow never expands regardless of available space
type: gotcha
module: core
created: 2026-04-07
---

# flexGrow is silently ignored when container has only one processed child

The flex layout engine in `flex.ts` skips the `flexGrow` distribution calculation entirely when there is only one processable child (or zero). The guard `numProcessedChildren > 1` means single-child containers with `flexGrow` set on the child see no expansion — no error, no warning, no layout change.

## Source

```typescript
// flex.ts
if (totalFlexGrow > 0 && numProcessedChildren > 1) {
  // ... distribute available space proportionally among flexGrow children ...
}
```

The guard is deliberate — with only one child, there are no other children competing for space, and the calculation "distribute remaining space proportionally" would be trivially applied to 100% of the available space. However, the result is still silence rather than full-width expansion.

## What counts as "processed"

`numProcessedChildren` is the length of `processableChildrenIndices`, which excludes:
- Children with `flexItem === false`
- Text nodes (`isTextNode(c)`)
- Element-text nodes without explicit dimensions (`isElementText(c)` with text but no width/height — these cause early return)

So if a container has two children but one has `flexItem={false}`, only one is "processed" and `flexGrow` is still skipped.

## Example

```tsx
// This does NOT work — only one processed child, flexGrow ignored
<view width={800} height={60} display="flex">
  <view flexGrow={1} height={60} />   {/* expects to fill 800px, actually stays at 0px */}
</view>

// This works — two children, flexGrow distributes correctly
<view width={800} height={60} display="flex">
  <view width={200} height={60} />
  <view flexGrow={1} height={60} />   {/* fills remaining 600px */}
</view>

// Workaround for single-child case — use explicit width or NaN fill behavior
<view width={800} height={60} display="flex">
  <view width={NaN} height={60} />    {/* NaN fills to parent width */}
</view>
```

## Workaround

For a single child that should fill its container:
- Use `width={NaN}` (or omit width) which triggers the fill-parent behavior via `element node NaN width and height default to parent dimensions minus position offset`
- Or explicitly set `width` to the container's fixed width
- Or add a second sibling with `width={0}` to satisfy the `> 1` guard (fragile; not recommended)

---

Related Entries:
- [[flexGrow distributes positive available space proportionally among children]] — the general flexGrow entry; does not document this single-child edge case
- [[element node NaN width and height default to parent dimensions minus position offset]] — the alternative fill-parent mechanism that works for single children
- [[flexItem false excludes a child from flex layout calculations]] — relevant because flexItem=false children reduce the processed count, potentially triggering this gotcha

Domains:
- [[styling guide]]
- [[core guide]]
