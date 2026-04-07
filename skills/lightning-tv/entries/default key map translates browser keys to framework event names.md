---
description: The built-in keyMapEntries maps 9 browser key names to semantic event names (Left, Right, Up, Down, Enter, Last, Space, Back, Escape) used to construct handler names like onLeft, onCaptureEnter
type: api
module: core
created: 2026-04-07
---

# default key map translates browser keys to framework event names

The focus manager maintains a `keyMapEntries` object that converts raw browser key identifiers into framework event names. These event names are used to derive the handler property names called on elements during propagation.

| Browser Key | Framework Event |
|-------------|-----------------|
| `ArrowLeft`  | `Left`  |
| `ArrowRight` | `Right` |
| `ArrowUp`    | `Up`    |
| `ArrowDown`  | `Down`  |
| `Enter`      | `Enter` |
| `l`          | `Last`  |
| ` ` (space)  | `Space` |
| `Backspace`  | `Back`  |
| `Escape`     | `Escape`|

From a mapped event name like `Left`, the framework constructs:
- Bubble handler: `onLeft`
- Bubble release handler: `onLeftRelease`
- Capture handler: `onCaptureLeft`
- Capture release handler: `onCaptureLeftRelease`

Keys not in the map are still propagated through `onKeyPress`/`onKeyHold` fallback handlers, but no specific event handler is called.

Users can override or extend this map via the `userKeyMap` option in [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]]. Setting a value to `null` removes that key from the map entirely. Array values let multiple browser keys map to the same event name.

Since [[focus manager propagates key events in capture then bubble phase]], the mapped event name is the mechanism connecting raw input to typed handler methods. The `Left`/`Right` event names map to `onLeft`/`onRight` — the exact handlers wired by [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]]. Similarly `Up`/`Down` map to the handlers in [[Column component is a vertical navigable list with flexDirection column and 30px gap]].

---

Related Entries:
- [[EventHandlers type generates key handler props from key map interfaces]] — the TypeScript type that auto-generates the handler prop names from these event name strings

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
