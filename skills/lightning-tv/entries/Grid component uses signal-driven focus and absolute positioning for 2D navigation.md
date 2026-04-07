---
description: Grid manages focus with a createSignal internally, positions items at absolute coordinates computed from columns and item dimensions, and scrolls the container by animating y
type: architecture
module: primitives
created: 2026-04-07
---

# Grid component uses signal-driven focus and absolute positioning for 2D navigation

Grid is architecturally distinct from [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]]. It does not use flex layout or the `withScrolling` engine. Instead:

1. **Reactive signal for focus**: `const [focusedIndex, setFocusedIndex] = createSignal(0)` — all navigation updates this signal
2. **Absolute positioning**: Each item receives explicit `x` and `y` coordinates computed from its index and `columns`
3. **Vertical scroll only**: The grid container's `y` position is a `createMemo` derived from `focusedIndex`

```tsx
<Grid
  items={myItems}
  columns={4}
  itemWidth={200}
  itemHeight={200}
  itemOffset={20}
  scroll="auto"
  looping={false}
  selected={0}
  onSelectedChanged={(index, grid, elm) => { ... }}
>
  {({ item, index, x, y, width, height }) => (
    <view x={x} y={y} width={width} height={height}>
      {item.title}
    </view>
  )}
</Grid>
```

## Item Positioning

Items are positioned by the Grid, not flex:
```ts
x = (index % columns) * (itemWidth + itemOffset)
y = Math.floor(index / columns) * (itemHeight + itemOffset)
```

`itemOffset` adds spacing between items (applies to both axes equally).

## Scroll Calculation

```ts
const scrollY = createMemo(() =>
  props.scroll === "none"
    ? props.y ?? 0
    : -Math.floor(focusedIndex() / columns()) * totalHeight() + (props.y || 0)
);
```

The entire focused row is scrolled to `y=0` (plus the `y` offset). Scrolling is whole-row only — no fractional row scrolling. Animated via `transition={{ y: true }}`.

## strictBounds={false}

Grid renders with `strictBounds={false}` so the renderer allows item content to exceed the container's logical bounds. This is required since absolute-positioned items at the bottom of the grid extend beyond the container's computed height.

## Render-Prop Children

Grid uses a render-prop pattern with Solid's `Index` (not `For`):
```tsx
<Index each={props.items}>
  {(item, index) => (
    <props.children item={item()} index={index} width={...} height={...} x={...} y={...} />
  )}
</Index>
```

`Index` keys by array position, not item identity. Reordering items causes updates to existing elements, not remounts. Since [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]], item identity doesn't matter for the grid's focus state — only the index position does.

## Selected Prop Sync

A `createEffect` watches `props.selected` reactively:
```ts
createEffect(() => {
  if (props.selected !== undefined && props.items?.length > props.selected) {
    moveFocus(props.selected - currentIndex);
  }
});
```
This enables external control: change `selected` signal and the grid navigates to match.

Grid registers its directional key handlers directly on the grid element. These participate in [[focus manager propagates key events in capture then bubble phase]] — returning `true` from a handler consumes the event, returning `false` lets it bubble to parent containers. Because Grid uses an internal `focusedIndex` signal rather than the `selected` prop after mount, [[plinko prop transfers the current row selected index to the next row on vertical navigation]] does not work with Grid children — `moveSelection` assigns to `active.selected`, but Grid's `createEffect` won't observe that as a reactive update to its signal.

---

Related Entries:
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — the virtual counterpart; VirtualGrid uses flex layout instead of absolute positioning, making it more memory-efficient for large datasets
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — Grid items typically use $focus styles to highlight the selected cell; the focusedIndex signal drives which item has focus
- [[animatable number properties route through transition system before reaching the renderer]] — Grid animates its y position via `transition={{ y: true }}` to scroll rows into view
- [[Grid looping wraps to the same column in the first or last row not to item index 0]] — extends this by documenting the looping behavior
- [[Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus]] — programmatic navigation gotcha for unfocused grids

Source: [[primitives-Grid]]
Domains:
- [[navigation guide]]
- [[components guide]]
