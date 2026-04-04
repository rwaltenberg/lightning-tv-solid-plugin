# Wrap Mode Uses First Child's Cross-Size for All Lines

> In wrapped layouts, the cross-size (line height/width) of the first child is used as a uniform line size for every line.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

The wrap algorithm does not track per-line cross sizes. Instead, it captures the first processable child's cross-size and uses that value uniformly for all lines.

From `flex.ts` line 219 / `flexLayout.ts` line 302:
```ts
const childCrossSizeVar =
  numProcessedChildren > 0 ? childCrossSizes[0]! : containerCrossSize;
```

This value is used as the line height (for row direction) when advancing the cross position at each line break: `crossPos += childCrossSizeVar + crossGap`.

If children have varying cross-sizes, the lines will not adapt to their tallest item. All lines are spaced by the first child's cross-size. Items taller than the first child will overflow into the next line's space, causing overlap.

## Code Example

```tsx
// First child is 30px tall. All lines will be spaced 30px apart.
// Second child is 80px tall -- it will overlap the next line.
<view display='flex' width={250} flexDirection='row' flexWrap='wrap' gap={10}>
  <view width={100} height={30} />   {/* cross-size anchor = 30 */}
  <view width={100} height={80} />   {/* overflows 30px line, overlaps next line */}
  <view width={100} height={30} />
</view>
```

## Gotchas

- This is not configurable. There is no per-line cross-size tracking regardless of `alignItems` setting.
- The workaround is to ensure all children in a wrapped layout have the same cross-size, or to fix the first child's cross-size to the largest expected value.
- `alignContent` does not exist in this engine, so there is also no way to distribute space between lines after the fact. See `no-align-content.md`.

## Related Notes

- [flex/no-align-content.md] -- no space distribution between lines
- [flex/wrap-reverse-broken.md] -- `wrap-reverse` bug in new engine
