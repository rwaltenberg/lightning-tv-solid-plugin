---
description: spatialHandleNavigation is a key handler for flex-wrap containers that moves within rows on horizontal keys and finds the nearest cross-axis child on vertical keys
type: api
module: primitives
created: 2026-04-07
---

# spatialHandleNavigation navigates flex-wrap containers by finding the closest child in the next row or column

`spatialHandleNavigation` is a `KeyHandler` for navigating flex-wrap containers (not Grid, but flex containers that wrap naturally). It handles all four arrow keys differently based on whether the movement is along the flex axis or the cross axis.

```tsx
import { spatialHandleNavigation } from '@lightningtv/solid/primitives';

<view
  display="flex"
  flexWrap="wrap"
  onUp={spatialHandleNavigation}
  onDown={spatialHandleNavigation}
  onLeft={spatialHandleNavigation}
  onRight={spatialHandleNavigation}
  selected={0}
  onSelectedChanged={(idx, el, child, lastIdx) => { ... }}
>
```

## Along the flex axis (same row/column)

For horizontal movement in a row-direction flex container:
- Scans forward/backward from current child
- Stops when a child has a different cross-axis position (different row)
- Selects the next non-`skipFocus` child in the same row

```ts
if (child[crossDir] !== prevChild[crossDir]) break; // different row
return selectChild(this as NavigableElement, i);
```

## Across the flex axis (change row/column)

For vertical movement in a row-direction container:
- Scans through all children in the direction
- Skips same-row children
- Among different-row children, finds the one with the smallest flex-axis distance to the current child
- Stops scanning when distance starts increasing (greedy proximity search)

```ts
const distance = Math.abs(child[flexDir] - prevChild[flexDir]);
if (distance >= closestDist) break; // getting further away
```

## Direction detection

Uses `this.flexDirection` to determine which axis is the flex axis:
```ts
const flexDir = this.flexDirection === 'column' ? 'y' : 'x';
const crossDir = flexDir === 'x' ? 'y' : 'x';
```

Works for both `flexDirection: 'row'` (default) and `flexDirection: 'column'` containers.

## Contrast with Grid

Since [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]], Grid computes positions mathematically from `columns`. `spatialHandleNavigation` uses actual element positions from the renderer — making it work for any flex-wrap layout, including non-uniform item sizes.

The handlers participate in [[focus manager propagates key events in capture then bubble phase]] — returning `true` from any directional handler stops propagation to parent containers. This follows the same [[KeyHandlerReturn true stops key event propagation up the focus chain]] contract as Row/Column's handlers.

---

Related Entries:
- [[spatialForwardFocus selects the geometrically closest child when a container receives focus]] — companion handler for initial focus placement; spatialForwardFocus decides WHICH child gets focus first, spatialHandleNavigation handles movement after
- [[flexWrap wraps children to new rows when they overflow the main axis]] — provides the layout foundation that spatialHandleNavigation is designed to navigate

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
