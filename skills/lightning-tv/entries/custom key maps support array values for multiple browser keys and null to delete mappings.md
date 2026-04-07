---
description: When passing userKeyMap to useFocusManager, array values map multiple browser keys to one event, null removes an existing mapping, and string values add or override a single key
type: pattern
module: core
created: 2026-04-07
---

# custom key maps support array values for multiple browser keys and null to delete mappings

The `flattenKeyMap` function processes user-provided key maps into the internal `keyMapEntries` / `keyHoldMapEntries` objects. Its behavior differs from a simple merge.

## Merge rules

| Value type | Behavior |
|------------|----------|
| `string`   | `targetMap[value] = key` — maps the browser key (`key`) to the event name (`value`) |
| `string[]` | Each string in the array maps that browser key to the same event; allows multiple physical keys to trigger one event |
| `null`     | `delete targetMap[key]` — removes the mapping for that browser key entirely |

```typescript
useFocusManager({
  userKeyMap: {
    // Single override — map 'k' key to 'Back' event
    k: 'Back',
    // Array — both numpad enter and regular enter trigger 'Enter'
    NumpadEnter: 'Enter',
    // Remove Backspace from triggering 'Back'
    Backspace: null,
  }
});
```

## Gotcha: key/value inversion

The `userKeyMap` format has **event name as value, browser key as object key**. Internally, `flattenKeyMap` inverts this: `targetMap[eventName] = browserKey`. This is the opposite of what a naive object merge would produce and is why `flattenKeyMap` is needed at all.

Because [[default key map translates browser keys to framework event names]] applies globally, changes made via `userKeyMap` affect all subsequent key events for the lifetime of the focus manager — there is no scoped per-component key map.

---
Source: [[core-focusManager]]
Domains:
- [[focus guide]]
