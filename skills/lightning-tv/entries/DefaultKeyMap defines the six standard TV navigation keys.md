---
description: DefaultKeyMap maps six logical key names — Left, Right, Up, Down, Enter, Last — to keyboard key names or codes; each accepts a single value or array for multi-key binding
type: api
module: core
created: 2026-04-07
---

# DefaultKeyMap defines the six standard TV navigation keys

`DefaultKeyMap` is the foundation of the key input system. It maps logical direction/action names to physical keyboard keys or codes.

## Interface

```typescript
interface DefaultKeyMap {
  Left: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Right: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Up: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Down: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Enter: KeyNameOrKeyCode | KeyNameOrKeyCode[];
  Last: KeyNameOrKeyCode | KeyNameOrKeyCode[];
}

type KeyNameOrKeyCode = string | number;
```

## The six keys

| Logical name | Typical mapping | Notes |
|---|---|---|
| `Left` | `'ArrowLeft'` | D-pad left |
| `Right` | `'ArrowRight'` | D-pad right |
| `Up` | `'ArrowUp'` | D-pad up |
| `Down` | `'ArrowDown'` | D-pad down |
| `Enter` | `'Enter'` | OK/Select button |
| `Last` | `'Backspace'` | Back/Previous button |

## Multi-key binding

Each entry accepts an array to map multiple physical keys to one logical action:
```typescript
Left: ['ArrowLeft', 'KeyA'],  // both trigger 'Left' action
```

## KeyMap extension

`KeyMap extends DefaultKeyMap` adds `[key: string]: KeyNameOrKeyCode | KeyNameOrKeyCode[]` for custom application-specific keys beyond the 6 defaults.

## How these become handler props

Since [[EventHandlers type generates key handler props from key map interfaces]], the six keys automatically become `onLeft`, `onRight`, `onUp`, `onDown`, `onEnter`, `onLast` plus their Release and Capture variants on every element node.

---

Related Entries:
- [[EventHandlers type generates key handler props from key map interfaces]] — transforms this map into JSX handler props
- [[DefaultKeyHoldMap adds hold-key detection for EnterHold]] — the companion interface for hold events

Source: [[core-focusKeyTypes]]
Domains:
- [[focus guide]]
