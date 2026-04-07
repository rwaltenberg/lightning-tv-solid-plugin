---
description: Row renders a flex-row view with display:flex and gap:30 by default, wired for left/right key navigation and scrolling on selection change
type: api
module: primitives
created: 2026-04-07
---

# Row component is a horizontal navigable list with built-in flex layout and 30px gap

Row wraps a `<view>` element with default styles `{ display: 'flex', gap: 30 }` and wires up complete horizontal navigation: `onLeft`/`onRight` key handlers, `forwardFocus` via [[navigableForwardFocus selects the child at the selected index when a container receives focus]], and automatic `scrollRow` on every selection change.

```tsx
import { Row } from '@lightningtv/solid/primitives';

<Row selected={0} scroll="auto" onSelectedChanged={(idx, row, child) => console.log(idx)}>
  <Item />
  <Item />
  <Item />
</Row>
```

Row accepts all `NavigableProps`:

| Prop | Type | Notes |
|------|------|-------|
| `selected` | `number` | Initial selected index; defaults to 0 |
| `scroll` | `'always' \| 'none' \| 'edge' \| 'auto' \| 'center'` | Scroll mode; see [[withScrolling implements six scroll modes for navigable list containers]] |
| `scrollIndex` | `number` | Index at which auto-scrolling begins |
| `offset` | `number` | Adjusts starting x position of the list |
| `plinko` | `boolean` | Transfers child's selected index to next row on vertical exit |
| `wrap` | `boolean` | Wraps navigation back to the beginning |
| `onSelectedChanged` | `OnSelectedChanged` | Fires on every selection change, including first focus |
| `onScrolled` | `function` | Fires when scroll position changes |
| `onLeft` / `onRight` | `KeyHandler` | User-provided handlers chained before internal handler |

User `style` is merged with built-in Row styles via `combineStyles(props.style, RowStyles)` — user values take precedence.

Since [[Row and Column apply direction-specific transitions on each navigation key press]], the scroll animation automatically matches the direction of movement.

Since [[flex layout engine positions children using justifyContent along the main axis]], Row's built-in `justifyContent:'flexStart'` means the container auto-resizes to its children by default — set `flexBoundary:'fixed'` to hold a fixed width.

Since [[dollar-prefix state keys in NodeStyles apply style variants based on active states]], items inside Row commonly define `$focus` styles to visually indicate selection.

Row's `onLeft`/`onRight` handlers participate in [[focus manager propagates key events in capture then bubble phase]] — they fire in the bubble phase, so a parent container can intercept with `onCaptureLeft`/`onCaptureRight` before Row processes the key. Since [[moveSelection skips skipFocus children and supports plinko index transfer between rows]], Row's directional key handlers internally call moveSelection to advance selection.

---

Related Entries:
- [[Row and Column use @once annotations so handler props cannot be updated after mount]] — critical gotcha: Row's event handler props are frozen at render time

Source: [[primitives-Row]]
Domains:
- [[navigation guide]]
- [[components guide]]
