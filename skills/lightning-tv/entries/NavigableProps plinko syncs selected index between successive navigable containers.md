---
description: plinko=true sets the next Row or Column's selected index to match the previous container's index; used for 2D grid navigation where column position should persist across rows
type: pattern
module: primitives
created: 2026-04-07
---

# NavigableProps plinko syncs selected index between successive navigable containers

The `plinko` prop on `NavigableProps` enables a TV UI navigation pattern where navigating between rows (or columns) preserves the horizontal (or vertical) position:

```ts
/** Plinko - sets the selected item of the next row to match the previous row */
plinko?: boolean;
```

Without plinko: navigating down from any position in row A always lands on `selected=0` (or whatever the target row's current selected is) in row B.

With plinko: navigating down from position 3 in row A sets position 3 in row B before focus transfers. This creates the "plinko" effect where the focus drops straight down like a plinko ball.

This is applied per-container on `NavigableElement` (Row/Column). Both the source and the target containers need appropriate setup — the source reads its current `selected`, and the navigation handler applies it to the next container's `selected` before entering.

Since [[NavigableProps wrap prop enables circular navigation within Row or Column children]] is a different navigation modifier, these can be combined: a row with both `wrap` and `plinko` would wrap within itself but preserve position when navigating out.

The `onSelectedChanged` callback accompanies plinko-driven changes — it fires with `lastSelectedIndex` when the index changes, enabling custom logic on position sync.

Since [[moveSelection skips skipFocus children and supports plinko index transfer between rows]], plinko is implemented inside `moveSelection` before `selectChild` runs — the destination container's `selected` is updated before [[navigableForwardFocus selects the child at the selected index when a container receives focus]] reads it, ensuring the right child gets focus on entry.

---

Related Entries:
- [[plinko prop transfers the current row selected index to the next row on vertical navigation]] — the implementation-level entry with the actual `moveSelection` code

Source: [[primitives-types]]
Domains:
- [[navigation guide]]
- [[components guide]]
