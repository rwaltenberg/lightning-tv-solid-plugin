# VirtualGrid

> Virtualized grid component that windows rows from a large data array using flexWrap layout.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`VirtualGrid` virtualizes a grid by slicing items into a visible window based on cursor row. It uses `flexWrap: 'wrap'` style with `transition: { y: true }`. Horizontal navigation uses `navigableHandleNavigation` (standard Row behavior). Vertical navigation moves by `columns` items using custom `onUp`/`onDown` handlers. Uses `columnScroll` (from `withScrolling(false)`) for vertical scroll positioning. Exposes `scrollToIndex(index)`. Calls `onEndReached` when cursor exceeds threshold.

**Export**: `export function VirtualGrid<T>(props: VirtualGridProps<T>): JSX.Element`

## Props / API

```ts
type VirtualGridProps<T> = NewOmit<RowProps, 'children'> & {
  each: readonly T[] | undefined | null | false;
  columns: number;
  rows?: number;             // visible rows, default 1
  buffer?: number;           // default 2
  onEndReached?: () => void;
  onEndReachedThreshold?: number;
  children: (item: Accessor<T>, index: Accessor<number>) => JSX.Element;
};
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `each` | `readonly T[] \| undefined \| null \| false` | required | Data array |
| `columns` | `number` | required | Number of columns in the grid |
| `rows` | `number` | `1` | Number of visible rows |
| `buffer` | `number` | `2` | Extra rows to render beyond visible rows |
| `onEndReached` | `() => void` | — | Callback when cursor exceeds threshold |
| `onEndReachedThreshold` | `number` | — | How far from end to trigger onEndReached |
| `children` | `(item: Accessor<T>, index: Accessor<number>) => JSX.Element` | required | Render function |

Also accepts all `RowProps` (except `children`) via `NewOmit<RowProps, 'children'>`.

**Internal layout**: `flexWrap: 'wrap'` style (NOT absolute positioning). Vertical scroll uses `columnScroll` from `withScrolling(false)`.

## Gotchas

- `columns` is required (not optional unlike `Grid` which defaults to 4).
- Children receive `Accessor<T>` and `Accessor<number>` -- must call `item()` to get value.
- Horizontal navigation uses standard Row `navigableHandleNavigation`.
- Vertical navigation moves by `columns` count, not by 1.
- Uses `flexWrap: 'wrap'` unlike `Grid` which uses absolute positioning.
- DO NOT mix with static JSX children.

## Related Notes

- [api/grid.md] -- non-virtualized grid with absolute positioning
- [api/virtual-row.md] -- horizontal virtualized list
- [api/virtual-column.md] -- vertical virtualized list
- [api/scrolling.md] -- scroll utilities
