---
description: Fetches pages via a `(page: number) => Promise<T[]>` fetcher, accumulating results into a single `items` array; detects end automatically when a fetch returns empty
type: api
module: primitives
created: 2026-04-07
---

`createInfiniteItems` is a data primitive for infinite-scroll patterns. It wraps `createResource` to accumulate paginated results and signals when all data has been fetched:

```ts
const [items, { page, setPage, end, setEnd, setItems }] = createInfiniteItems(fetcher);
```

## Signature

```ts
function createInfiniteItems<T>(
  fetcher: (page: number) => Promise<T[]>,
): [Accessor<T[]>, { page, setPage, setItems, end, setEnd }]
```

## How It Works

```ts
const [page, setPage] = createSignal(0);
const [contents] = createResource(page, fetcher);
createComputed(() => {
  const content = contents();
  if (!content) return;
  batch(() => {
    if (content.length === 0) setEnd(true);
    setItems(p => [...p, ...content]);
  });
});
```

- `createResource(page, fetcher)`: fetches automatically when `page` changes
- Results are appended, not replaced: `[...prev, ...content]`
- An empty response automatically sets `end = true`
- `batch` ensures `end` and `items` update atomically (no intermediate renders)

## Usage with Virtual Components

The canonical pattern pairs with Virtual's `onEndReached`:

```ts
const [items, { setPage, end }] = createInfiniteItems(async (page) => {
  return (await api.getItems({ page })).results;
});

<VirtualRow
  each={items()}
  onEndReachedThreshold={5}
  onEndReached={() => !end() && setPage(p => p + 1)}
>
  {(item) => <Card item={item()} />}
</VirtualRow>
```

Since [[Virtual onEndReached fires when cursor moves within threshold of the last item]], calling `setPage` triggers a new fetch that appends to the array, seamlessly extending the virtual list.

## Manual Controls

| Control | Use |
|---------|-----|
| `setItems` | Reset items array (e.g., on search query change) |
| `setEnd` | Manually signal no more data |
| `setPage` | Trigger next fetch |
| `end()` | Guard against fetching past the end |

## Note on IntersectionObserver

The comment in the source notes this is adapted from `@solid-primitives/pagination` but without IntersectionObserver (unavailable in Lightning TV). Page advancement must be triggered explicitly — typically via `onEndReached`.

---

Source: [[primitives-createInfiniteItems]]
Domains:
- [[performance guide]]
- [[components guide]]
