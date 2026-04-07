---
description: `cursor` is the index in the full dataset; `selected` is the position within the rendered slice — they must stay synchronized on every navigation
type: architecture
module: primitives
created: 2026-04-07
---

Virtual components maintain two independent position signals that mean different things:

- **`cursor`**: absolute index in the full `each` array (0 to `itemCount - 1`)
- **`selected`**: position within the currently rendered slice (0 to `displaySize + bufferSize - 1`)

Both are needed because the underlying Row/Column navigable operates only in terms of rendered children. It knows `selected` (which child is focused). The Virtual layer needs `cursor` to know where in the full dataset the user actually is.

## How They Diverge

When `scroll: 'edge'`, the selection moves from 0 to `displaySize` before the window shifts. During this phase, `selected` increments while `cursor` also increments but the window start does not change. Once the window starts shifting, `selected` stays pinned at `displaySize` while `cursor` continues to grow.

When `scroll: 'always'`, `selected` stays pinned at `bufferSize` while `cursor` freely advances — the window always follows.

## Update Protocol

`onSelectedChanged` fires whenever the underlying Row/Column changes its selected child. It computes `delta` (direction of movement), updates `cursor` via `setCursor`, recomputes the slice via `computeSlice`, and then manually corrects `elm.selected` to the new position within the slice:

```ts
const newState = computeSlice(cursor(), delta, slice());
setSlice(newState);
elm.selected = newState.selected;
```

The underlying Row/Column's own `selected` tracking is overridden here — Virtual takes full control.

## Gotcha: Slice Index Correction

After the window shifts, `this.selected` inside the Row/Column refers to the old rendered index. Virtual must correct it to point to the right item in the new slice. Missing this step causes focus to land on the wrong item after a window shift.

---

Related Entries:
- [[withScrolling implements six scroll modes for navigable list containers]] — the scroll mode chosen on the underlying Row/Column determines how cursor and selected diverge; 'always' pins selected, 'edge' lets it move
- [[Virtual uniformSize caches first item dimension to avoid repeated DOM reads]] — window shift calculations depend on the item size derived from the selected index

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
