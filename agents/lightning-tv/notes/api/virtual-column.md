# VirtualColumn

> Virtualized vertical list that only renders a windowed slice of a large data array.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`VirtualColumn` is the vertical equivalent of `VirtualRow`. It shares the same `VirtualProps<T>` interface and the same internal windowing behavior. It only renders `displaySize + bufferSize` items at any time using `@solid-primitives/list`'s `<List>` component for efficient keyed rendering. It maintains a `cursor` signal tracking the logical position in the full data array, computes a `slice` of the data to render, and supports all scroll modes. Implements adaptive animation duration for rapid navigation. Handles `wrap` mode with modular arithmetic. Calls `onEndReached` when cursor approaches threshold. `uniformSize` (default `true`) caches item sizes. Exposes `scrollToIndex(index)` which resets animations and repositions.

Children receive `Accessor<T>` and `Accessor<number>` for index.

**Export**: `export function VirtualColumn<T>(props: VirtualProps<T>): JSX.Element`

## Props / API

```ts
type VirtualProps<T> = NewOmit<RowProps, 'children'> & {
  each: readonly T[] | undefined | null | false;
  displaySize: number;
  bufferSize?: number;       // default 2
  wrap?: boolean;
  scrollIndex?: number;
  onEndReached?: () => void;
  onEndReachedThreshold?: number;
  debugInfo?: boolean;
  factorScale?: boolean;
  uniformSize?: boolean;     // default true
  children: (item: Accessor<T>, index: Accessor<number>) => JSX.Element;
};
```

Same props as `VirtualRow`. The vertical navigation uses `transitionUp` (300ms) and `transitionDown` (300ms) inherited from `ColumnProps` behavior.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `each` | `readonly T[] \| undefined \| null \| false` | required | Data array |
| `displaySize` | `number` | required | Number of items visible at once |
| `bufferSize` | `number` | `2` | Extra items to render beyond displaySize |
| `wrap` | `boolean` | — | Circular list with modular arithmetic |
| `scrollIndex` | `number` | — | Index at which scrolling begins (auto mode) |
| `onEndReached` | `() => void` | — | Callback when cursor approaches end |
| `onEndReachedThreshold` | `number` | — | How far from end to trigger onEndReached |
| `debugInfo` | `boolean` | — | Enable debug overlay |
| `factorScale` | `boolean` | — | Account for scaled item sizes in scroll calculations |
| `uniformSize` | `boolean` | `true` | Cache item sizes after first measurement |
| `children` | `(item: Accessor<T>, index: Accessor<number>) => JSX.Element` | required | Render function |

## Gotchas

- `displaySize` is required. Without it, slice calculations produce incorrect results.
- Children receive `Accessor<T>` -- you MUST call `item()` not `item.title`.
- Both `item` and `index` are `Accessor`s (unlike LazyColumn where `index` is a raw number).
- `scrollToIndex` resets animations and repositions.
- DO NOT mix with static JSX children.

## Related Notes

- [api/virtual-row.md] -- horizontal equivalent; same VirtualProps interface
- [api/virtual-grid.md] -- virtualized grid
- [api/lazy-column.md] -- progressive rendering alternative
- [api/create-infinite-items.md] -- pagination primitive
- [api/scrolling.md] -- scroll mode behavior
