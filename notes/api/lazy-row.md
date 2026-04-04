# LazyRow

> Progressively rendering horizontal list that adds items incrementally to avoid frame drops.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`LazyRow` progressively renders items rather than all at once. It starts with 0 (or `upCount` if `sync`) items and incrementally adds one per frame (~60fps via `setTimeout 16ms`). When `eagerLoad` is true, continues loading items beyond `upCount` using `scheduleTask` (idle-time loading). On navigation toward unloaded items, increments the offset to render the next item. `delay` adds throttled loading -- if user navigates faster than the delay, falls back to synchronous loading. `buffer` determines how many items ahead of selection to pre-render. Wraps a `Row` via `Dynamic` component. Exposes `lazyScrollToIndex(index)` which pre-loads enough items then delegates to the inner `scrollToIndex`.

**Export**: `export function LazyRow<T extends readonly any[]>(props: LazyProps<T>): JSX.Element`

## Props / API

```ts
type LazyProps<T extends readonly any[]> = NewOmit<NodeProps, 'children'> & {
  each: T | undefined | null | false;
  upCount: number;
  buffer?: number;
  delay?: number;
  sync?: boolean;
  eagerLoad?: boolean;
  noRefocus?: boolean;
  children: (item: Accessor<T[number]>, index: number) => JSX.Element;
};
```

| Prop | Type | Description |
|------|------|-------------|
| `each` | `T \| undefined \| null \| false` | Data array |
| `upCount` | `number` | Initial number of items to render synchronously |
| `buffer` | `number` | Items ahead of selection to pre-render |
| `delay` | `number` | Throttle delay for loading; navigating faster falls back to sync loading |
| `sync` | `boolean` | Start with `upCount` items rendered synchronously |
| `eagerLoad` | `boolean` | Continue loading beyond `upCount` using `scheduleTask` (idle time) |
| `noRefocus` | `boolean` | Suppress refocus after items load |
| `children` | `(item: Accessor<T[number]>, index: number) => JSX.Element` | Render function |

**Note on children signature**: `item` is an `Accessor` but `index` is a raw `number` (unlike VirtualRow where both are Accessors).

## Gotchas

- `item` is an `Accessor` -- must call `item()`. But `index` is a raw `number`, not an Accessor (contrast with VirtualRow).
- DO NOT mix with static JSX children. LazyRow expects `each` array + render function.
- `lazyScrollToIndex` pre-loads items before delegating to inner Row's `scrollToIndex`.
- Progressive rendering uses `setTimeout 16ms` per item -- not all items are available immediately.

## Related Notes

- [api/lazy-column.md] -- vertical equivalent with same LazyProps interface
- [api/row.md] -- Row that LazyRow wraps via Dynamic
- [api/virtual-row.md] -- alternative virtualization strategy
