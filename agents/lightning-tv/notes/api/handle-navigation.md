# handleNavigation API

> handleNavigation(direction) factory, moveSelection(el, delta), navigableHandleNavigation, selectChild(), wrap and plinko behavior.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

Located in `src/primitives/utils/handleNavigation.ts`.

**Exports:**

```ts
// Key Handlers
export function handleNavigation(
  direction: 'up' | 'right' | 'down' | 'left'
): KeyHandler;

export const navigableHandleNavigation: KeyHandler;
export const spatialHandleNavigation: KeyHandler;

// Selection
export function moveSelection(el: NavigableElement, delta: number): boolean;

// Default Transitions
export const defaultTransitionBack: { x: { duration: 180; easing: string } };
export const defaultTransitionForward: { x: { duration: 180; easing: string } };
export const defaultTransitionDown: { y: { duration: 300; easing: string } };
export const defaultTransitionUp: { y: { duration: 300; easing: string } };
```

**`handleNavigation(direction)`**: Factory function returning a `KeyHandler` that always moves selection by -1 (left/up) or +1 (right/down) and also applies directional transitions. Use this for clarity and transition support when attaching to specific directional handler props.

**`navigableHandleNavigation`**: A single `KeyHandler` that reads `e.key` to determine direction at call time. Can handle both directions from one handler. Do not attach to a single directional handler prop like `onLeft` — it works (since `e.key` is always `ArrowLeft` in that context) but `handleNavigation('left')` is preferred for clarity.

**`spatialHandleNavigation`**: Key handler for flex-wrap containers. Behavior:
1. Determines flex direction (`flexDirection === 'column'` -> primary axis is `y`; default is `x`).
2. Maps the arrow key to movement along the flex axis or cross axis.
3. Flex-axis movement: Scans forward/backward among siblings sharing the same cross-axis position (same row/column).
4. Cross-axis movement: Finds the closest child in the next row/column by measuring distance along the flex axis from the current child's position.

**`moveSelection(el, delta)`**: The workhorse of list navigation. Steps:
1. Starting from `el.selected + delta`, scan children in the direction of `delta`.
2. Skip children with `skipFocus`.
3. If `wrap` is enabled, indices wrap around modulo `children.length`.
4. If no valid child is found AND the current selection is either out-of-bounds, has `skipFocus`, or is already focused, return `false` (let the event bubble to the parent).
5. If `plinko` is enabled, copy the `selected` index from the current child to the target child before focusing it.
6. Call `selectChild()` to apply focus and fire `onSelectedChanged`.

**`selectChild()`**: Sets `selected`, calls `child.setFocus()`, and fires `onSelectedChanged`.

**`NavigableElement` interface:**

```ts
interface NavigableProps extends NodeProps {
  onSelectedChanged?: OnSelectedChanged;
  scroll?: 'always' | 'none' | 'edge' | 'auto' | 'center';
  scrollIndex?: number;
  selected?: number;
  offset?: number;
  plinko?: boolean;
  wrap?: boolean;
  onScrolled?: (elm: NavigableElement, offset: number, isInitial: boolean) => void;
  transitionUp?: NodeStyles['transition'];
  transitionDown?: NodeStyles['transition'];
  transitionLeft?: NodeStyles['transition'];
  transitionRight?: NodeStyles['transition'];
}

interface NavigableElement extends ElementNode, NavigableProps {
  selected: number;
  scrollToIndex: (this: NavigableElement, index: number) => void;
}
```

**Deprecated:**
```ts
/** @deprecated Use navigableForwardFocus instead */
export function onGridFocus(cb?: OnSelectedChanged): ForwardFocusHandler;
```

## Gotchas

- `navigableHandleNavigation` reads `e.key` at call time — attaching it to `onLeft` works but `handleNavigation('left')` is preferred for clarity and transition support.
- `moveSelection` returns `false` when no valid child is found, allowing the event to bubble to the parent container.
- `wrap` wraps around modulo `children.length` — combining with `plinko` can produce unexpected index copies if row lengths differ.

## Related Notes

- [focus/skip-focus.md] -- moveSelection skips skipFocus children
- [focus/plinko-navigation.md] -- plinko behavior in moveSelection
- [api/forward-focus-handlers.md] -- navigableForwardFocus and spatialForwardFocus that pair with these handlers
- [focus/spatial-navigation.md] -- spatialHandleNavigation usage guide
