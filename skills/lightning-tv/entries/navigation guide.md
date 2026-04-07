---
description: Navigation patterns — Row/Column directional nav, spatial navigation, scrolling modes, plinko and wrap behaviors
type: moc
---

# navigation guide

Navigation in Lightning TV/Solid handles how focus moves between items in lists and grids. Includes linear navigation (Row/Column), spatial navigation (Grid), multiple scroll modes, and special behaviors like plinko and wrap.

## Core Concepts

### Linear Navigation Components
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — the primary horizontal list, wired for left/right keys with auto-scrolling and flex layout defaults
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — the vertical counterpart, using up/down keys with a 300ms transition (longer than Row's 180ms)

### Grid Navigation
- [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]] — fundamentally different from Row/Column: uses reactive signals, absolute item positioning, and whole-row scrolling
- [[Grid looping wraps to the same column in the first or last row not to item index 0]] — looping preserves column alignment, unlike Row/Column's sequential wrap
- [[Grid onSelectedChanged has a different signature than NavigableProps onSelectedChanged]] — critical API mismatch: argument count and order differ from NavigableProps

### Focus Forwarding
- [[navigableForwardFocus selects the child at the selected index when a container receives focus]] — the standard forward focus handler for Row/Column; reads `selected`, skips skipFocus children
- [[spatialForwardFocus selects the geometrically closest child when a container receives focus]] — geometric alternative that selects the child closest to the previously active element
- [[forwardFocus is required on containers that hold focusable children]] — custom container views must set forwardFocus explicitly; Row and Column wire it automatically

### Key Handlers
- [[moveSelection skips skipFocus children and supports plinko index transfer between rows]] — the core movement engine; handles skipFocus traversal and plinko state transfer
- [[spatialHandleNavigation navigates flex-wrap containers by finding the closest child in the next row or column]] — key handler for flex-wrap layouts without an explicit Grid

### Scroll Engine
- [[withScrolling implements six scroll modes for navigable list containers]] — `auto`, `edge`, `always`, `center`, `bounded`, and `none`; scrollRow and scrollColumn are the built exports
- [[scrollIndex prop delays auto scroll start until selected item reaches the threshold index]] — creates a non-scrolling zone at the start of the list in auto mode
- [[bounded scroll mode freezes the list when the last upCount items are in focus]] — implementation-only mode not in public types; stops list movement in the end zone
- [[edge scroll mode uses renderer viewport state to detect off-screen items before scrolling]] — uses renderer's renderState===8 (InViewPort) to decide when to scroll
- [[withScrolling caches screen offset and target position to handle mid-animation scroll correctly]] — internal `_screenOffset`, `_targetPosition`, `_initialPosition` cache on the element node

### Transitions
- [[Row and Column apply direction-specific transitions on each navigation key press]] — merges `transitionLeft/Right/Up/Down` into `el.transition` on each keypress

### Special Behaviors
- [[plinko prop transfers the current row selected index to the next row on vertical navigation]] — enables straight-line up/down navigation through a Column of Rows
- [[NavigableProps plinko syncs selected index between successive navigable containers]] — the prop-level definition: sets next container's selected to match prior container's index before focus transfers
- [[NavigableProps wrap prop enables circular navigation within Row or Column children]] — wraps from last to first child; all-skipFocus children with wrap cause infinite loop
- [[NavigableProps transitionLeft right up down override animation per navigation direction]] — direction-specific transition overrides taking precedence over itemTransition

## Patterns

### Column of Rows with Plinko
The canonical TV grid pattern: a `Column` containing `Row` elements, with `plinko={true}` on the Column so vertical navigation preserves horizontal position. Since [[plinko prop transfers the current row selected index to the next row on vertical navigation]], focus moves "straight down" through the grid.

```tsx
<Column plinko={true}>
  <Row scroll="auto">{ /* items */ }</Row>
  <Row scroll="auto">{ /* items */ }</Row>
</Column>
```

### Programmatic Navigation
Use `scrollToIndex` on the element ref to jump to a specific item. Note the behavioral differences between Row/Column and Grid — see [[Row and Column scrollToIndex imperatively sets selected scrolls and focuses without respecting scroll=none]] and [[Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus]].

### Scroll Mode Selection
- Content longer than screen, standard behavior: `scroll="auto"`
- Large list, keep first items visible longer: `scroll="auto" scrollIndex={2}`
- Focus stays fixed, list scrolls: `scroll="always"`
- Center focused item on screen: `scroll="center"`
- Hold until boundary reached: `scroll="edge"`
- Custom controlled position: `scroll="none"`

## Gotchas

- [[Row and Column use @once annotations so handler props cannot be updated after mount]] — all handler props on Row/Column are frozen at render via `/* @once */`
- [[Row onLayout only chains scrollRow when selected prop is non-zero at mount]] — `selected={0}` (or undefined) never triggers initial scroll
- [[onSelectedChanged fires on every focus including first focus not only on changes]] — cannot use `index !== lastIndex` as a reliable change detector
- [[Row and Column scrollToIndex imperatively sets selected scrolls and focuses without respecting scroll=none]] — `scroll='none'` does not prevent programmatic scrollToIndex
- [[Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus]] — calling scrollToIndex on an unfocused Grid steals focus to the grid
- [[NavigableProps wrap prop enables circular navigation within Row or Column children]] — all-skipFocus children with `wrap=true` cause infinite loop in findFirstFocusableChildIdx
- [[withScrolling caches screen offset and target position to handle mid-animation scroll correctly]] — `_screenOffset` is computed once; external position changes after first scroll make it stale
- [[bounded scroll mode freezes the list when the last upCount items are in focus]] — `bounded` mode is not in the public NavigableProps type
- [[selectedNode getter mutates selected index as a side effect]] — reading selectedNode advances this.selected past any non-element nodes (e.g., text nodes injected by SolidJS); not a pure read

## Open Questions
- How does the scroll engine handle containers inside a clipped parent that later resizes?
- What happens when `plinko` is used with a Grid child — does Grid's internal signal accept the selected assignment?
- Is there a way to reset `_screenOffset` without remounting the component?
