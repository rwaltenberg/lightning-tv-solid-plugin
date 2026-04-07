---
description: EventHandlers<Map> is a mapped type that auto-generates on*, on*Release, and onCapture* prop names for every key in a key map, plus onCaptureKey and onCaptureKeyRelease wildcards
type: architecture
module: core
created: 2026-04-07
---

# EventHandlers type generates key handler props from key map interfaces

`EventHandlers<Map>` is the type-level mechanism that transforms a key map interface (like `DefaultKeyMap`) into the corresponding handler props. It is applied twice in `NodeProps` — once for the default key map and once for the hold key map.

## The type definition

```typescript
type EventHandlers<Map> = {
  [K in keyof Map as `on${Capitalize<string & K>}`]?: KeyHandler;
} & {
  [K in keyof Map as `on${Capitalize<string & K>}Release`]?: KeyHandler;
} & {
  [K in keyof Map as `onCapture${Capitalize<string & K>}`]?: KeyHandler;
} & {
  onCaptureKey?: KeyHandler;
  onCaptureKeyRelease?: KeyHandler;
};
```

## Generated props for DefaultKeyMap

From the 6 default keys (Left, Right, Up, Down, Enter, Last), EventHandlers generates:

| Pattern | Example props |
|---------|---------------|
| `on{Key}` | `onLeft`, `onRight`, `onUp`, `onDown`, `onEnter`, `onLast` |
| `on{Key}Release` | `onLeftRelease`, `onRightRelease`, `onEnterRelease`, etc. |
| `onCapture{Key}` | `onCaptureLeft`, `onCaptureRight`, `onCaptureEnter`, etc. |
| Wildcards | `onCaptureKey`, `onCaptureKeyRelease` |

## Generated props for KeyHoldMap

From `DefaultKeyHoldMap` containing `EnterHold`:
- `onEnterHold`, `onEnterHoldRelease`, `onCaptureEnterHold`

## Capture vs bubble phase

`onCapture*` handlers fire during the capture phase (root → leaf), `on*` handlers fire during the bubble phase (leaf → root). Because of this since [[FocusNode interface defines the focus lifecycle callbacks]], parent containers can intercept navigation before it reaches the focused leaf.

## KeyHandler signature

```typescript
type KeyHandler = (
  this: ElementNode,      // node with the handler
  e: KeyboardEvent,       // raw keyboard event
  target: ElementNode,    // currently focused element
  handlerElm: ElementNode, // same as this
  mappedEvent?: string,   // 'Left', 'Enter', etc.
) => KeyHandlerReturn;    // boolean | void — true stops propagation
```

---

Related Entries:
- [[DefaultKeyMap defines the six standard TV navigation keys]] — the key map that drives default prop generation
- [[FocusNode interface defines the focus lifecycle callbacks]] — the FocusNode onKeyPress/onKeyHold are the raw-key equivalents
- [[KeyHandlerReturn true stops key event propagation up the focus chain]] — explains the return value

Source: [[core-focusKeyTypes]]
Domains:
- [[focus guide]]
