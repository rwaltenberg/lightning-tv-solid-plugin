# Call useFocusManager Exactly Once

> useFocusManager() must be called exactly once at the app root. Duplicate calls register duplicate keydown/keyup listeners causing double-handling of every key event.

**Source**: `spatial-navigation.md` | **Severity**: critical

## Detail

The core `useFocusManager` appends `keydown`/`keyup` listeners to `document` every time it is called. Calling it multiple times will register duplicate listeners, causing double-handling of every key event.

Call it exactly once at the top level of the application, inside the SolidJS reactive root. This must happen before any element attempts to take focus.

The SolidJS version (`src/primitives/useFocusManager.ts`) wraps the core engine and:
1. Captures the current SolidJS `owner` via `getOwner()`.
2. Binds `Config.setActiveElement` to update the SolidJS `activeElement` signal within that owner.
3. Calls the core `useFocusManagerCore` with the SolidJS `runWithOwner` context, ensuring all key handlers execute inside SolidJS's reactive tracking.
4. Creates an effect that mirrors the core focus path into a reactive SolidJS signal.
5. Registers `onCleanup` to remove `keydown`/`keyup` listeners and clear hold timeouts.

The `flattenKeyMap` function modifies the module-level `keyMapEntries` object. Once `useFocusManager` has been called, changing the key map requires calling `useFocusManager` again — but that violates the single-call rule. Design the key map fully before the single initialization call.

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

- Calling `useFocusManager` more than once registers duplicate `keydown`/`keyup` listeners on `document`, causing every key event to be handled twice.
- The key map cannot be safely changed after initialization without violating the single-call rule.
- Must be called before any element attempts to take focus.

## Related Notes

- [api/use-focus-manager.md] -- full API including KeyMap interface and KeyHoldOptions
- [focus/default-key-map.md] -- default key mappings and flattenKeyMap behavior
- [focus/two-phase-propagation.md] -- the propagation model that the focus manager drives
