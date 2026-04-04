# VirtualRow

> Virtualized horizontal list that only renders a windowed slice of a large data array.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`VirtualRow` only renders `displaySize + bufferSize` items at any time. It uses `@solid-primitives/list`'s `<List>` component for efficient keyed rendering. It maintains a `cursor` signal tracking the logical position in the full data array. It computes a `slice` of the data to render based on scroll type, cursor position, and buffer. Supports all scroll modes: `'auto'`, `'always'`, `'edge'`, `'none'`. Implements adaptive animation duration -- rapid navigation shortens transition durations. Handles `wrap` mode with modular arithmetic for circular lists. Calls `onEndReached` when cursor approaches the threshold from the end. The `uniformSize` prop (default `true`) caches item sizes after first measurement for performance. `factorScale` accounts for scaled item sizes during scroll calculations. Exposes `scrollToIndex(index)` which resets animations and repositions.

Children receive `Accessor<T>` (not raw `T`) and `Accessor<number>` for index.

**Export**: `export function VirtualRow<T>(props: VirtualProps<T>): JSX.Element`

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

Also accepts all `RowProps` (except `children`): `scroll`, `selected`, `offset`, `plinko`, `onSelectedChanged`, `onScrolled`, `transitionLeft`, `transitionRight`, etc.

## Code Example

```tsx
import { VirtualRow } from '@lightningtv/solid/primitives';

<VirtualRow
  each={items()}
  displaySize={7}
  bufferSize={2}
  scroll="auto"
  onEndReached={() => loadMoreItems()}
  onEndReachedThreshold={10}
>
  {(item, index) => <Card title={item().title} />}
</VirtualRow>
```

Infinite scrolling with `createInfiniteItems`:

```tsx
import { createInfiniteItems, VirtualRow } from '@lightningtv/solid/primitives';

const [items, { setPage, end }] = createInfiniteItems((page) => fetchItems(page));

<VirtualRow
  each={items()}
  displaySize={7}
  onEndReached={() => !end() && setPage(p => p + 1)}
  onEndReachedThreshold={5}
>
  {(item) => <Card title={item().title} />}
</VirtualRow>
```

## Gotchas

- `displaySize` is required. Without it, slice calculations produce incorrect results.
- Children receive `Accessor<T>` -- you MUST call `item()` not `item.title`. Accessing `item.title` directly is wrong.
- Both `item` and `index` are `Accessor`s (unlike LazyRow where `index` is a raw number).
- `scrollToIndex` resets animations and repositions from scratch.
- DO NOT mix with static JSX children. VirtualRow expects `each` array + render function only.

## Related Notes

- [api/virtual-column.md] -- vertical equivalent
- [api/virtual-grid.md] -- virtualized grid
- [api/lazy-row.md] -- progressive rendering alternative
- [api/create-infinite-items.md] -- pagination primitive for onEndReached pattern
- [api/scrolling.md] -- scroll mode behavior
