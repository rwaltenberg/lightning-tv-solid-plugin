# Row

> Horizontal flex navigation container that handles left/right key events, scrolling, and focus forwarding.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

`Row` renders a horizontal flex container. It intercepts `onLeft` and `onRight` key events for navigation between children. It calls `scrollRow` on `onSelectedChanged` (unless `scroll === 'none'`). It sets `forwardFocus` to `navigableForwardFocus` so that when the Row receives focus, it forwards to its `selected` child. It exposes `scrollToIndex(index)` on the element node. If `props.selected` is set, hooks `onLayout` to call `scrollRow` for initial positioning.

**Export**: `export const Row: Component<RowProps>`

**Default Styles Applied**:
```ts
{ display: 'flex', gap: 30 }
```

**Directional Transition Defaults**: `transitionLeft` (180ms cubic-bezier back) and `transitionRight` (180ms cubic-bezier forward).

## Props / API

From `types.ts`:

```ts
type OnSelectedChanged = (
  this: NavigableElement,
  selectedIndex: number,
  elm: NavigableElement,
  active: ElementNode,
  lastSelectedIndex?: number,
) => void;

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

interface NavigableStyleProperties {
  scrollIndex?: number;
  itemSpacing?: NodeStyles['gap'];
  itemTransition?: NodeStyles['transition'];
}

interface RowProps extends NavigableProps, NavigableStyleProperties {
  onLeft?: KeyHandler;
  onRight?: KeyHandler;
}
```

| Prop | Type | Description |
|------|------|-------------|
| `selected` | `number` | Initial selected child index (default `0`) |
| `scroll` | `'always' \| 'none' \| 'edge' \| 'auto' \| 'center'` | Scroll behavior |
| `scrollIndex` | `number` | Index at which scrolling begins (for `auto` mode) |
| `offset` | `number` | Adjust the x position offset |
| `plinko` | `boolean` | Propagate selected index to next row's children |
| `wrap` | `boolean` | Wrap navigation from last to first child |
| `onLeft` | `KeyHandler` | Custom left handler (chained before default) |
| `onRight` | `KeyHandler` | Custom right handler (chained before default) |
| `onSelectedChanged` | `OnSelectedChanged` | Callback on selection change |
| `onScrolled` | `(elm, offset, isInitial) => void` | Callback after scroll position change |
| `transitionLeft` | `NodeStyles['transition']` | Override left-nav transition |
| `transitionRight` | `NodeStyles['transition']` | Override right-nav transition |

## Code Example

```tsx
import { Row } from '@lightningtv/solid/primitives';

<Row scroll="auto" selected={0}>
  <Card />
  <Card />
  <Card />
</Row>
```

Programmatic navigation:

```tsx
let rowRef;
<Row ref={rowRef} scroll="auto">
  {items.map(item => <Card />)}
</Row>

// Later:
rowRef.scrollToIndex(5);
```

## Gotchas

- DO NOT put non-focusable children in Row without `skipFocus` -- they will receive focus and break navigation.
- DO NOT mutate `selected` directly on the element (`rowRef.selected = 5`). This bypasses `onSelectedChanged`, breaks scroll, and breaks focus. Use `rowRef.scrollToIndex(5)` instead.
- DO NOT return `true` from `onSelectedChanged`. It is NOT part of the key handler chain. Only `onLeft`/`onRight` key handlers support `return true` to consume an event.
- DO NOT set `scroll="none"` and expect `onScrolled` to fire. When `scroll` is `'none'`, the scroller returns immediately and `onScrolled` will never be invoked.
- User-provided `onLeft`/`onRight` handlers run FIRST via `chainFunctions`. Return `true` to suppress default navigation.

## Related Notes

- [api/column.md] -- vertical equivalent of Row
- [api/virtual-row.md] -- virtualized version of Row
- [api/lazy-row.md] -- progressively rendered version of Row
- [api/scrolling.md] -- all scroll modes used by Row
