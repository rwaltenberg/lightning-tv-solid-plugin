---
description: FocusNode provides three focus lifecycle callbacks — onFocus, onBlur, onFocusChanged — each receiving the current and previous focused elements with the node as this context
type: api
module: core
created: 2026-04-07
---

# FocusNode interface defines the focus lifecycle callbacks

The `FocusNode` interface declares three callbacks that any element node can implement to respond to focus state transitions. These are woven into `NodeProps` via the type composition chain.

## onFocus

Called when focus enters this node's subtree for the first time (or returns to it):

```typescript
onFocus?(
  this: ElementNode,                    // the node with the callback
  currentFocusedElm: ElementNode,       // element that now has focus
  prevFocusedElm: ElementNode | undefined, // previously focused (undefined on first)
  nodeWithCallback: ElementNode,        // same as this
): void
```

## onBlur

Called when focus leaves this node or its subtree:

```typescript
onBlur?(
  this: ElementNode,
  currentFocusedElm: ElementNode,       // element that has focus NOW
  prevFocusedElm: ElementNode,          // element that HAD focus (NOT optional here)
  nodeWithCallback: ElementNode,
): void
```

Note: `prevFocusedElm` is NOT optional in `onBlur`, unlike in `onFocus`. Both params are guaranteed to be defined.

## onFocusChanged

Called on both focus enter AND leave — a unified callback for both transitions:

```typescript
onFocusChanged?(
  this: ElementNode,
  hasFocus: boolean,                    // true = gained, false = lost
  currentFocusedElm: ElementNode,
  prevFocusedElm: ElementNode | undefined,
  nodeWithCallback: ElementNode,
): void
```

Use `hasFocus` to distinguish entering (`true`) from leaving (`false`).

## onKeyPress and onKeyHold on FocusNode

`FocusNode` also declares generic key handlers for catching any key press/hold without routing through the key map:

```typescript
onKeyPress?(
  this: ElementNode,
  e: KeyboardEvent,
  mappedKeyEvent: string | undefined,  // 'Left', 'Enter', etc. or undefined
  handlerElm: ElementNode,
  currentFocusedElm: ElementNode,
): KeyHandlerReturn  // boolean | void
```

These complement the auto-generated `EventHandlers<DefaultKeyMap>` (onLeft, onRight, etc.) by catching unmapped keys.

---

Related Entries:
- [[EventHandlers type generates key handler props from key map interfaces]] — the auto-generated directional key handlers
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — FocusNode is mixed into NodeProps
- [[ElementNode is the primary abstraction for all renderable elements in Lightning TV Solid]] — the runtime class that implements FocusNode; every element on screen can declare these callbacks
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — the runtime mechanism that fires these callbacks during focus path changes

Source: [[core-focusKeyTypes]]
Domains:
- [[focus guide]]
