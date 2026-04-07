---
description: useMouse registers window listeners for wheel, click, and mousedown, then performs BFS hit testing on the ElementNode tree using absX/absY coordinates scaled by deviceLogicalPixelRatio
type: api
module: primitives
created: 2026-04-07
---

# useMouse adds pointer support via BFS hit testing over the Lightning element tree

Lightning TV renders via WebGL, so elements are not DOM nodes and receive no native pointer events. `useMouse` bridges this gap by hit-testing the ElementNode tree on pointer events.

```typescript
export function useMouse<TApp extends ElementNode = ElementNode>(
  myApp: TApp = rootNode as TApp,
  throttleBy: number = 100,     // ms for hover position throttle
  options?: UseMouseOptions,     // optional custom states
): void
```

## Hit Testing: `getChildrenByPosition`

BFS traversal from `myApp` root. At each level, nodes are filtered to those that:
1. Are `ElementNode` (not text nodes)
2. Have `alpha !== 0` (visible)
3. Do NOT have `skipFocus` set
4. Pass AABB collision: `absX * DPR <= px <= (absX + width) * DPR` and same for Y

When multiple siblings overlap at the same position, the one with the highest `zIndex` is selected. The result is an ordered array from root ancestor down to the deepest matching child.

DPR scaling uses `Config.rendererOptions?.deviceLogicalPixelRatio` (defaults to 1).

## Two Operational Modes

**Focus mode (no `options.customStates`):**
- On hover: walks the BFS result looking for an element with `onEnter`, `onMouseClick`, `onFocus`, or `Config.focusStateKey`. Found element's `setFocus()` is called.
- On click: uses `findElementByActiveElement` — checks if the currently active element is under the click, then walks parent chain looking for a parent with `onMouseClick`.

**Custom states mode (`options.customStates` provided):**
- On hover: finds element with `hoverState` custom state marker, applies state to it, removes from previous.
- On click: finds element with `hoverState`, triggers click handling.
- On mousedown: finds element with `hoverState`, applies `pressedState` to it; removed on click/mouseup.

## Click Handling Priority

When a click lands on an element, `handleElementClick` resolves it in this order:
1. If `onMouseClick` handler exists → call `onMouseClick(event, element)`
2. Else if `onEnter` handler exists → call `onEnter()`
3. Else → `element.setFocus()` + dispatch synthetic `Enter` keydown + keyup (1ms apart)

The synthetic Enter dispatch means that clicking on any focusable element that has keyboard Enter behavior will work via mouse without special mouse handling.

## Scroll Translation

```typescript
const handleScroll = throttle((e: WheelEvent) => {
  if (e.deltaY < 0) document.body.dispatchEvent(createKeyboardEvent('ArrowUp', 38));
  else if (e.deltaY > 0) document.body.dispatchEvent(createKeyboardEvent('ArrowDown', 40));
  // 250ms after last scroll event: dispatch keyup for both
}, 250);
```

Wheel events are translated to `ArrowUp`/`ArrowDown` keyboard events, enabling the standard key navigation system to handle scrolling. The 250ms keyup delay simulates key release.

## `forwardStates` Traversal

Both hover detection and click detection walk up the parent chain while `parent.forwardStates` is set. This ensures that when a child element has the hover state, the parent (e.g., a card container) receives the action instead of the child label.

## Row/Column `selected` Sync

On hover, if the hovered element's parent has a `selected` property (Row/Column navigation containers), the parent's `selected` index is updated. This keeps keyboard and mouse navigation state in sync.

Since [[forwardStates propagates parent state array to all children on every state change]], `useMouse` respects the same forwarding mechanism for consistency with the focus/state system.

In focus mode, hover calls `element.setFocus()` which flows through [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — meaning mouse hover triggers the same `onFocus`/`onBlur` callbacks and state changes as keyboard navigation, keeping both interaction modes in sync.

BFS traversal operates entirely over [[ElementNode is the primary abstraction for all renderable elements in Lightning TV Solid]] instances — text nodes are explicitly skipped (`NodeType.TextNode` check) since they are not interactive hit targets.

---

Source: [[primitives-useMouse]]
Domains:
- [[input guide]]
