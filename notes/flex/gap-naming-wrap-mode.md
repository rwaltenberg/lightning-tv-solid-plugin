# Gap Naming is Counterintuitive in Row Wrap Mode

> In a row+wrap layout, `columnGap` controls the vertical gap between wrapped lines -- not the horizontal gap between columns.

**Source**: `flex-layout-engine.md` | **Severity**: important

## Detail

In the wrap code path, the cross-axis gap is resolved as:
```ts
const crossGap = isRow ? (node.columnGap ?? gap) : (node.rowGap ?? gap);
```

For a `row` direction layout:
- **Main-axis (horizontal) gap** between items on the same line: `gap`
- **Cross-axis (vertical) gap** between wrapped lines: `columnGap ?? gap`

This means `columnGap` controls the vertical spacing between wrapped rows in a `row` layout. `rowGap` is not used for `row` direction.

For a `column` direction layout, the cross-axis gap uses `rowGap ?? gap`.

The test at `flex.spec.ts` line 469-507 confirms this: in a row layout with `columnGap: 10`, the vertical spacing between wrapped lines uses `columnGap` (10), not `rowGap`.

## Code Example

```tsx
// In a row+wrap layout, set columnGap to control VERTICAL space between lines
<view display='flex' width={250} height={200}
      flexDirection='row' flexWrap='wrap'
      gap={5}        // horizontal gap between items on the same line
      columnGap={20} // vertical gap between wrapped lines (counterintuitive!)
>
  <view width={100} height={50} />
  <view width={100} height={50} />
  <view width={100} height={50} />
</view>
```

## Gotchas

- `rowGap` is unused in `row` direction. Setting it has no effect on line spacing.
- `columnGap` is unused in `column` direction. `rowGap` controls cross-axis spacing there.
- This naming is counterintuitive compared to CSS, where `row-gap` sets the gap between rows (i.e., the vertical gap in a row layout).
- This only applies inside the wrap code path. When `flexWrap` is not set, `gap` is the only spacing property used.

## Related Notes

- [flex/wrap-cross-size-first-child.md] -- the other wrap spacing concern (line height based on first child)
- [flex/wrap-reverse-broken.md] -- `wrap-reverse` does not activate the wrap path in the new engine
