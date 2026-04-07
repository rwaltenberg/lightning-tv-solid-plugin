---
description: `onEndReached` callback fires on every `onSelectedChanged` event once `cursor >= itemCount - onEndReachedThreshold`; pairs with createInfiniteItems for pagination
type: api
module: primitives
created: 2026-04-07
---

VirtualRow and VirtualColumn support an infinite-scroll pattern through `onEndReached` and `onEndReachedThreshold`:

```ts
onEndReached?: () => void;
onEndReachedThreshold?: number;
```

## How It Fires

Inside `onSelectedChanged` (called on every navigation event):

```ts
if (
  props.onEndReachedThreshold !== undefined &&
  cursor() >= itemCount() - props.onEndReachedThreshold
) {
  props.onEndReached?.();
}
```

The check runs after updating `cursor`, so it reflects the new position. The callback fires every time the cursor is within `threshold` of the end — not just once. Callers must guard against redundant fetches (e.g., checking an `end` signal).

## Threshold Semantics

`onEndReachedThreshold: 5` means the callback fires when the user is 5 or fewer items from the last item. This gives the app time to fetch the next page before the user reaches the actual end.

## Integration with createInfiniteItems

The canonical pattern:

```ts
const [items, { setPage, end }] = createInfiniteItems(fetcher);

<VirtualRow
  each={items()}
  displaySize={6}
  onEndReachedThreshold={4}
  onEndReached={() => {
    if (!end()) setPage(p => p + 1);
  }}
>
  {(item) => <ItemCard item={item()} />}
</VirtualRow>
```

Since [[createInfiniteItems accumulates paginated data into a single reactive array]], incrementing the page triggers a fetch and the resulting items are appended to the array that `VirtualRow` reads.

## VirtualGrid Equivalent

VirtualGrid exposes the same `onEndReached` and `onEndReachedThreshold` props with identical semantics, checked in its own `onSelectedChanged`.

---

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
