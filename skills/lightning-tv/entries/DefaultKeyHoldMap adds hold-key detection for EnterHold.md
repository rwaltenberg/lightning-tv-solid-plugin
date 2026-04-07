---
description: DefaultKeyHoldMap defines EnterHold as the only built-in hold event; KeyHoldMap extends it for custom hold keys; KeyHoldOptions configures the hold threshold in milliseconds
type: api
module: core
created: 2026-04-07
---

# DefaultKeyHoldMap adds hold-key detection for EnterHold

The hold-key system is defined separately from the regular key map. `DefaultKeyHoldMap` provides one built-in hold event (`EnterHold`) which fires when the Enter key is held past a configurable threshold.

## DefaultKeyHoldMap

```typescript
interface DefaultKeyHoldMap {
  EnterHold: KeyNameOrKeyCode | KeyNameOrKeyCode[];
}
```

`KeyHoldMap extends DefaultKeyHoldMap` — open for adding custom hold events.

## Generated handler props

Since [[EventHandlers type generates key handler props from key map interfaces]], `EventHandlers<KeyHoldMap>` applied to `NodeProps` generates:
- `onEnterHold` — fires when Enter is held past threshold
- `onEnterHoldRelease` — fires when held Enter is released
- `onCaptureEnterHold` — capture-phase variant

## KeyHoldOptions

Configuration for the hold detection system:
```typescript
type KeyHoldOptions = {
  userKeyHoldMap: Partial<KeyHoldMap>; // custom hold key bindings
  holdThreshold?: number;              // milliseconds before hold fires
};
```

The `holdThreshold` is optional — the focus manager has a default value (not defined in this type file).

## Extending with custom hold events

```typescript
// In app setup
const myKeyHoldMap: KeyHoldMap = {
  EnterHold: 'Enter',  // default
  BackHold: 'Backspace', // custom: hold back button
};
```

After extension, `onBackHold`, `onBackHoldRelease`, `onCaptureBackHold` become available on element nodes.

---

Related Entries:
- [[DefaultKeyMap defines the six standard TV navigation keys]] — the non-hold key system
- [[EventHandlers type generates key handler props from key map interfaces]] — transforms this map into JSX handler props
- [[key hold detection fires after 500ms and cancels if key is released early]] — the runtime mechanism that fires hold events for keys registered in this map
- [[useHold distinguishes quick press from long hold using a threshold timeout and wasHeld flag]] — per-component alternative when per-direction hold is needed without modifying the global hold map

Source: [[core-focusKeyTypes]]
Domains:
- [[focus guide]]
- [[input guide]]
