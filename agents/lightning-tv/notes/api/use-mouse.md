# useMouse

> Enables mouse/pointer input for TV apps with hover, click, and scroll wheel support.

**Source**: `ui-primitives.md` | **Severity**: informational

## Detail

`useMouse` enables mouse/pointer input for TV apps.

- **Hover**: Finds the deepest element at the pointer position via BFS, respecting zIndex. Calls `setFocus()` or applies a custom hover state.
- **Click**: Dispatches `onMouseClick` or `onEnter`, or synthesizes keyboard Enter events.
- **Scroll wheel**: Converts scroll wheel events to `ArrowUp`/`ArrowDown` keyboard events.

**Export**: `export function useMouse<TApp>(myApp?, throttleBy?, options?): void`

## Props / API

```ts
useMouse<TApp>(myApp?, throttleBy?, options?)
```

| Parameter | Description |
|-----------|-------------|
| `myApp` | Optional app reference |
| `throttleBy` | Optional throttle duration for hover events |
| `options` | Additional options |

**Hover behavior**: BFS traversal of Lightning scene graph by position, respecting zIndex ordering to find the deepest element under the pointer.

**Click behavior**: Dispatches `onMouseClick` event first, then `onEnter`, or synthesizes a keyboard Enter event.

**Scroll behavior**: Converts mouse wheel events to `ArrowUp`/`ArrowDown` keyboard events.

## Gotchas

- Hover uses BFS (breadth-first search) respecting zIndex -- the deepest matching element receives focus.

## Related Notes

- [api/use-hold.md] -- key hold/release detection utility
- [api/use-announcer.md] -- accessibility announcer
