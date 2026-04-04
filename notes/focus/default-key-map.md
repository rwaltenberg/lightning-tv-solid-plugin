# Default Key Map

> Default mappings from KeyboardEvent.key to event names, the KeyMap interface, and flattenKeyMap inversion behavior.

**Source**: `spatial-navigation.md` | **Severity**: informational

## Detail

The default key map is defined in `src/core/focusManager.ts`:

| `KeyboardEvent.key` | Mapped Event Name |
|---|---|
| `ArrowLeft` | `Left` |
| `ArrowRight` | `Right` |
| `ArrowUp` | `Up` |
| `ArrowDown` | `Down` |
| `Enter` | `Enter` |
| `l` | `Last` |
| `' '` (Space) | `Space` |
| `Backspace` | `Back` |
| `Escape` | `Escape` |

The key hold map is empty by default. The commented-out `Enter: 'EnterHold'` in the source shows the intended pattern for hold events.

Users supply custom mappings via the `KeyMap` interface:

```ts
interface DefaultKeyMap {
  Left: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Right: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Up: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Down: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Enter: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Last: KeyNameOrKeyCode | KeyNameOrKeyCode[];
}

interface KeyMap extends DefaultKeyMap {
  [key: string]: KeyNameOrKeyCode | KeyNameOrKeyCode[];
}
```

The `flattenKeyMap` function inverts the map: keys become event names, values become the raw key/keyCode triggers. An array value maps multiple physical keys to one event. A `null` value deletes an existing mapping.

`flattenKeyMap` modifies the module-level `keyMapEntries` object. This means the key map cannot be safely changed after `useFocusManager` has been called without violating the single-call rule.

## Code Example

```tsx
// Custom key mappings passed to useFocusManager
useFocusManager({ Menu: 'm', Play: ['p', 'MediaPlayPause'] });
```

## Gotchas

- The key hold map is empty by default — hold events require explicit configuration via `KeyHoldOptions`.
- `flattenKeyMap` modifies module-level state. Changing the key map after initialization is not safe.
- A `null` value in the user key map deletes an existing default mapping.
- Array values allow multiple physical keys to trigger the same event name.

## Related Notes

- [focus/use-focus-manager-once.md] -- key map must be finalized before the single useFocusManager call
- [api/use-focus-manager.md] -- KeyHoldOptions for hold event configuration
- [focus/two-phase-propagation.md] -- event names from the key map determine handler property names
