# scrollRow / scrollColumn (Scrolling Utilities)

> withScrolling utilities that translate selected-index changes into container axis translations.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`scrollRow` and `scrollColumn` are exported from `utils/withScrolling.ts`. They are called from `Row`'s and `Column`'s `onSelectedChanged` callbacks respectively. Scrolling is driven by changes to a `selected` index, NOT by pixel-based scroll positions. The utilities translate index changes into axis translations (x for Row, y for Column).

`NavigableElement.scrollToIndex(index)` is the imperative API to programmatically scroll to a specific index.

**Exports**: `export const scrollRow: Scroller`, `export const scrollColumn: Scroller`

## Props / API

### Scroll Modes

| Mode | Behavior |
|------|----------|
| `'auto'` | Scrolls item-by-item. Respects `scrollIndex` (the index at which scrolling begins). Stops when the last item is visible. |
| `'always'` | Selected item is always at the offset position. |
| `'edge'` | Only scrolls when the next element is outside the viewport. |
| `'center'` | Centers the selected item, clamped to bounds. |
| `'bounded'` | Like `'always'` but stops scrolling in the last `upCount` items. |
| `'none'` | No scrolling occurs. |

### NavigableElement.scrollToIndex

```ts
interface NavigableElement extends ElementNode, NavigableProps {
  selected: number;
  scrollToIndex: (this: NavigableElement, index: number) => void;
}
```

`scrollToIndex` is available on Row, Column, Grid, VirtualRow, VirtualColumn, VirtualGrid, LazyRow (as `lazyScrollToIndex`), LazyColumn (as `lazyScrollToIndex`).

## Gotchas

- DO NOT set `scroll="none"` and expect `onScrolled` to fire. When `scroll` is `'none'`, the scroller returns immediately and `onScrolled` will never be invoked.
- DO NOT mutate `selected` directly on an element. Use `scrollToIndex(index)` instead -- direct mutation bypasses `onSelectedChanged`, breaks scroll, and breaks focus.
- For VirtualRow/VirtualColumn, `scrollToIndex` also resets animations and repositions.
- For LazyRow/LazyColumn, the exposed method is `lazyScrollToIndex` (not `scrollToIndex`) -- it pre-loads enough items before delegating to the inner `scrollToIndex`.
- `'bounded'` mode stops scrolling in the last `upCount` items (similar to `'always'` elsewhere).
- `'auto'` mode respects the `scrollIndex` prop -- scrolling only begins when the selected index reaches or exceeds `scrollIndex`.

## Related Notes

- [api/row.md] -- uses scrollRow; accepts scroll prop
- [api/column.md] -- uses scrollColumn; accepts scroll prop
- [api/virtual-row.md] -- uses scrollRow via VirtualProps
- [api/virtual-column.md] -- uses scrollColumn via VirtualProps
- [api/virtual-grid.md] -- uses columnScroll for vertical scrolling
- [api/lazy-row.md] -- exposes lazyScrollToIndex
- [api/lazy-column.md] -- exposes lazyScrollToIndex
