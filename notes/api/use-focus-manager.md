# useFocusManager API

> Full API for useFocusManager. Core vs SolidJS versions, KeyMap interface, KeyHoldOptions (holdThreshold), focusPath signal, and cleanup.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

There are two versions of `useFocusManager`:

**SolidJS version** (`src/primitives/useFocusManager.ts`):

```ts
export const useFocusManager: (
  userKeyMap?: Partial<KeyMap>,
  keyHoldOptions?: KeyHoldOptions,
) => void;

// Also exports a reactive signal:
export const focusPath: Accessor<ElementNode[]>;
```

This version:
1. Captures the current SolidJS `owner` via `getOwner()`.
2. Binds `Config.setActiveElement` to update the SolidJS `activeElement` signal within that owner.
3. Calls the core `useFocusManagerCore` with the SolidJS `runWithOwner` context, ensuring all key handlers execute inside SolidJS's reactive tracking.
4. Creates an effect that mirrors the core focus path into a reactive SolidJS signal.
5. Registers `onCleanup` to remove `keydown`/`keyup` listeners and clear hold timeouts.

**Core version** (`src/core/focusManager.ts`):

```ts
export const useFocusManager: (options?: {
  userKeyMap?: Partial<KeyMap>;
  keyHoldOptions?: KeyHoldOptions;
  ownerContext?: (cb: () => void) => void;
}) => {
  cleanup: () => void;
  focusPath: () => ElementNode[];
};
```

The core version returns an object with `cleanup` and `focusPath`. The SolidJS version handles cleanup via `onCleanup` automatically and exports `focusPath` as a module-level `Accessor<ElementNode[]>`.

**KeyMap interface:**

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

**KeyHoldOptions:**

```ts
type KeyHoldOptions = {
  userKeyHoldMap: Partial<KeyHoldMap>;
  holdThreshold?: number; // default: 500ms
};

interface DefaultKeyHoldMap {
  EnterHold: KeyNameOrKeyCode | KeyNameOrKeyCode[];
}
```

The key hold system: on `keydown`, if the key is in the hold map, a timer starts at `holdThreshold` (default 500ms). If the key is released before the threshold, a normal keydown event fires. If the timer expires, a hold event fires instead. The key hold map is empty by default.

## Code Example

```tsx
import { useFocusManager } from '@lightningtv/solid/primitives';

function App() {
  useFocusManager();
  // or with custom keys:
  // useFocusManager({ Menu: 'm', Play: ['p', 'MediaPlayPause'] });
  return <MyAppContent />;
}
```

## Gotchas

- Call exactly once at app root — each call registers new `keydown`/`keyup` listeners on `document`.
- The key hold map is empty by default — hold events require explicit `KeyHoldOptions` with `userKeyHoldMap`.
- The `flattenKeyMap` function modifies module-level state; changing the key map after initialization is not safe.
- A `null` value in the user key map deletes an existing default mapping.
- The SolidJS version's `focusPath` is a module-level `Accessor`, not returned from the function call.

## Related Notes

- [focus/use-focus-manager-once.md] -- must be called exactly once
- [focus/default-key-map.md] -- default key mappings and flattenKeyMap behavior
- [focus/two-phase-propagation.md] -- propagation model driven by the focus manager
- [focus/focus-path.md] -- focusPath signal details
