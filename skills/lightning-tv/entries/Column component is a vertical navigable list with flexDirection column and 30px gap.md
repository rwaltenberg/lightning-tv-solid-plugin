---
description: Column renders a flex-column view with display:flex, flexDirection:column, and gap:30 by default, wired for up/down key navigation and scrolling
type: api
module: primitives
created: 2026-04-07
---

# Column component is a vertical navigable list with flexDirection column and 30px gap

Column is the vertical counterpart to [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]]. It applies `{ display: 'flex', flexDirection: 'column', gap: 30 }` as default styles and wires `onUp`/`onDown` key handlers with `scrollColumn` on selection change.

```tsx
import { Column } from '@lightningtv/solid/primitives';

<Column selected={0} scroll="edge" onSelectedChanged={(idx, col, child) => console.log(idx)}>
  <Item />
  <Item />
  <Item />
</Column>
```

Column accepts the same `NavigableProps` as Row but its directional key handlers are `onUp` and `onDown`. The key behavioral differences from Row:

| Aspect | Row | Column |
|--------|-----|--------|
| flexDirection | row (default) | `'column'` (explicit) |
| Key handlers | `onLeft` / `onRight` | `onUp` / `onDown` |
| Scroll function | `scrollRow` (x-axis) | `scrollColumn` (y-axis) |
| Transition duration | 180ms | 300ms |

Column's longer transition duration (300ms vs Row's 180ms) reflects the visual weight of vertical movement on TV interfaces.

Since [[withScrolling implements six scroll modes for navigable list containers]], Column supports the same `scroll` prop values as Row.

Since [[flexDirection controls main axis orientation and supports row-reverse and column-reverse]], Column's explicit `flexDirection:'column'` stacks children along the y-axis — vertical navigation aligns naturally with this axis.

Since [[dollar-prefix state keys in NodeStyles apply style variants based on active states]], items inside Column commonly define `$focus` styles to visually indicate the selected row.

Column's `onUp`/`onDown` handlers participate in [[focus manager propagates key events in capture then bubble phase]] — they fire in the bubble phase. Since [[moveSelection skips skipFocus children and supports plinko index transfer between rows]], Column's key handlers call moveSelection internally; with `plinko={true}`, moveSelection copies the departing Row's `selected` index to the arriving Row before focusing via [[navigableForwardFocus selects the child at the selected index when a container receives focus]].

---

Related Entries:
- [[plinko prop transfers the current row selected index to the next row on vertical navigation]] — Column-of-Rows plinko is the canonical pattern for TV grid layouts
- [[Row and Column use @once annotations so handler props cannot be updated after mount]] — critical gotcha: Column's event handler props are frozen at render time

Source: [[primitives-Column]]
Domains:
- [[navigation guide]]
- [[components guide]]
