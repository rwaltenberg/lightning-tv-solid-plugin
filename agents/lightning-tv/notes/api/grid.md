# Grid

> Absolute-positioned grid layout with focus management for static item arrays.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`Grid` renders all items using `<Index each={props.items}>` (keyed by index for stable references). It manages its own `focusedIndex` signal internally. Children are positioned absolutely using computed `x` and `y` based on column/row math -- Grid does NOT use flexbox. It scrolls by translating the container's `y` position. Handles `onUp`/`onDown` (vertical movement by `columns` count) and `onLeft`/`onRight` (horizontal within row). Supports `looping` for wrap-around in both axes. Uses `strictBounds={false}` on the container. Exposes `scrollToIndex(index)`.

**Export**: `export function Grid<T>(props: GridProps<T>): JSX.Element`

## Props / API

```ts
interface GridItemProps<T> {
  item:   T;
  index:  number;
  width:  number;
  height: number;
  x:      number;
  y:      number;
}

interface GridProps<T> extends NewOmit<NodeProps, 'children'> {
  items: readonly T[];
  children: (props: GridItemProps<T>) => JSX.Element;
  itemHeight?: number;   // default 300
  itemWidth?: number;    // default 300
  itemOffset?: number;   // gap between items
  columns?: number;      // default 4
  looping?: boolean;     // wrap navigation
  scroll?: "auto" | "none";
  selected?: number;
  onSelectedChanged?: (index: number, grid: ElementNode, elm?: ElementNode) => void;
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `items` | `readonly T[]` | required | Array of data items |
| `children` | `(props: GridItemProps<T>) => JSX.Element` | required | Render function |
| `itemHeight` | `number` | `300` | Height of each item |
| `itemWidth` | `number` | `300` | Width of each item |
| `itemOffset` | `number` | — | Gap between items |
| `columns` | `number` | `4` | Number of columns |
| `looping` | `boolean` | — | Wrap navigation in both axes |
| `scroll` | `"auto" \| "none"` | — | Scroll behavior |
| `selected` | `number` | — | Initial selected index |
| `onSelectedChanged` | `(index, grid, elm?) => void` | — | Callback on selection change |

## Code Example

```tsx
import { Grid } from '@lightningtv/solid/primitives';

<Grid items={myItems()} columns={4} itemWidth={300} itemHeight={200} itemOffset={20}>
  {(props) => (
    <view width={props.width} height={props.height} x={props.x} y={props.y}>
      <text>{props.item.title}</text>
    </view>
  )}
</Grid>
```

## Gotchas

- MUST apply `x`, `y`, `width`, `height` from `GridItemProps` to the item element. Grid uses absolute positioning, NOT flexbox. If you omit these, all items will stack at position 0,0.
- Uses `<Index>` (not `<For>`) for stable references keyed by index.
- `strictBounds={false}` is set on the container.
- `onUp`/`onDown` move by `columns` count, not by 1.

## Related Notes

- [api/virtual-grid.md] -- virtualized version for large datasets
- [api/row.md] -- horizontal navigation primitive
- [api/scrolling.md] -- scroll mode behavior
