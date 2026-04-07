---
description: Key handlers return boolean | void; returning true halts further propagation through ancestor nodes; falsy return (false, undefined, void) allows the event to continue bubbling
type: api
module: core
created: 2026-04-07
---

# KeyHandlerReturn true stops key event propagation up the focus chain

The return value of a key handler controls whether the event continues propagating up the focus chain to parent nodes.

## Type

```typescript
type KeyHandlerReturn = boolean | void;
```

## Behavior

| Return value | Effect |
|---|---|
| `true` | Stops propagation — no ancestor handlers fire |
| `false` | Allows propagation to continue |
| `void` / `undefined` | Allows propagation to continue |

## Pattern

This mirrors browser event propagation but uses return value instead of `event.stopPropagation()`.

```typescript
<View
  onLeft={(e, target, elm, mappedEvent) => {
    // Handle the navigation
    setSelectedIndex(i => i - 1);
    return true; // Prevent parent container from also handling Left
  }}
  onRight={(e, target) => {
    // Don't return true — let parent handle it too
    console.log('right pressed');
  }}
/>
```

## ForwardFocusHandler

`ForwardFocusHandler` uses the same return type:
```typescript
type ForwardFocusHandler = (this: ElementNode, elm: ElementNode) => boolean | void;
```
Returning a truthy value from `forwardFocus` indicates focus was successfully forwarded.

---

Related Entries:
- [[EventHandlers type generates key handler props from key map interfaces]] — these handlers use this return type
- [[FocusNode interface defines the focus lifecycle callbacks]] — onKeyPress/onKeyHold also use KeyHandlerReturn
- [[focus manager propagates key events in capture then bubble phase]] — the traversal loop that checks this return value to stop propagation
- [[moveSelection skips skipFocus children and supports plinko index transfer between rows]] — Row/Column's selection engine that returns true/false following this contract

Source: [[core-focusKeyTypes]]
Domains:
- [[focus guide]]
