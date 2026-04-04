# Column

> Vertical flex navigation container that handles up/down key events, scrolling, and focus forwarding.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`Column` is the vertical equivalent of `Row`. It renders a vertical flex container, intercepts `onUp`/`onDown` key events for navigation between children, and calls `scrollColumn` on `onSelectedChanged` (unless `scroll === 'none'`). It sets `forwardFocus` to `navigableForwardFocus` so that when the Column receives focus, it forwards to its `selected` child. It exposes `scrollToIndex(index)` on the element node. If `props.selected` is set, hooks `onLayout` to call `scrollColumn` for initial positioning.

**Export**: `export const Column: Component<ColumnProps>`

**Default Styles Applied**:
```ts
{ display: 'flex', flexDirection: 'column', gap: 30 }
```

**Directional Transition Defaults**: `transitionUp` (300ms) and `transitionDown` (300ms). Vertical movement is slower than horizontal Row transitions (180ms).

## Props / API

From `types.ts`:

```ts
interface NavigableStyleProperties {
  scrollIndex?: number;
  itemSpacing?: NodeStyles['gap'];
  itemTransition?: NodeStyles['transition'];
}

interface ColumnProps extends NavigableProps, NavigableStyleProperties {
  onDown?: KeyHandler;
  onUp?: KeyHandler;
}
```

Full `NavigableProps` (shared with Row):

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
```

## Code Example

```tsx
import { Column, Row } from '@lightningtv/solid/primitives';

<Column plinko scroll="auto">
  <Row scroll="auto">
    <Card /><Card /><Card />
  </Row>
  <Row scroll="auto">
    <Card /><Card /><Card />
  </Row>
</Column>
```

With `plinko`, vertical navigation between Rows maintains horizontal position. If Row A has `selected=3` and user presses Down, Row B will receive `selected=3` (clamped to its children length).

## Gotchas

- DO NOT put non-focusable children in Column without `skipFocus` -- they will receive focus and break navigation.
- DO NOT mutate `selected` directly on the element. Use `scrollToIndex(index)` instead.
- DO NOT return `true` from `onSelectedChanged`. Only `onUp`/`onDown` key handlers support `return true` to consume an event.
- DO NOT set `scroll="none"` and expect `onScrolled` to fire.
- User-provided `onUp`/`onDown` handlers run FIRST via `chainFunctions`. Return `true` to suppress default navigation.

## Related Notes

- [api/row.md] -- horizontal equivalent of Column
- [api/virtual-column.md] -- virtualized version of Column
- [api/lazy-column.md] -- progressively rendered version of Column
- [api/scrolling.md] -- all scroll modes used by Column
