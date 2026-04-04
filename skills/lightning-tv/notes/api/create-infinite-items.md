# createInfiniteItems

> Pagination primitive that fetches pages and appends to an accumulated items signal.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`createInfiniteItems` is a pagination primitive. It calls a `fetcher` function with a page number and appends the results to an accumulated items signal. Sets `end = true` when the fetcher returns an empty array.

**Export**:
```ts
export function createInfiniteItems<T>(
  fetcher: (page: number) => Promise<T[]>,
): [items: Accessor<T[]>, options: { page, setPage, setItems, end, setEnd }]
```

## Props / API

```ts
createInfiniteItems<T>(fetcher: (page: number) => Promise<T[]>)
```

Returns a tuple:
- `items`: `Accessor<T[]>` -- reactive accessor for the accumulated item array
- `options` object with:
  - `page`: current page signal
  - `setPage`: setter for page (incrementing triggers a new fetch)
  - `setItems`: manually set the items array
  - `end`: `Accessor<boolean>` -- true when fetcher returned empty
  - `setEnd`: manually set the end flag

## Code Example

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

- `end` is set to `true` automatically when the fetcher returns an empty array. Check `!end()` before calling `setPage` to avoid unnecessary fetches.
- Items are appended (not replaced) on each page fetch.

## Related Notes

- [api/virtual-row.md] -- primary consumer via onEndReached
- [api/virtual-column.md] -- can also use createInfiniteItems
