---
description: Focus management — capture/bubble event propagation, key press handling, throttling, and focus path tracking
type: moc
---

# focus guide

The focus system is the heart of TV app interaction. It manages 4-directional navigation, event propagation in capture-then-bubble phases, focus path tracking, and input throttling. All keyboard input flows through `useFocusManager` into `propagateKeyPress`, which walks the focus path in two ordered phases.

## Core Concepts

### Key Type System
- [[FocusNode interface defines the focus lifecycle callbacks]] — onFocus, onBlur, onFocusChanged signatures; parameter differences between onFocus (prevFocusedElm optional) and onBlur (not optional)
- [[EventHandlers type generates key handler props from key map interfaces]] — mapped type that auto-generates on*, on*Release, onCapture* props for every key in a key map
- [[DefaultKeyMap defines the six standard TV navigation keys]] — Left, Right, Up, Down, Enter, Last; each accepts string key name, numeric key code, or array of either
- [[DefaultKeyHoldMap adds hold-key detection for EnterHold]] — single built-in hold event; KeyHoldOptions configures holdThreshold in ms
- [[KeyHandlerReturn true stops key event propagation up the focus chain]] — returning true halts propagation; false/void/undefined allows bubbling

### Focus Manager
- [[focus path is an ordered array from focused leaf to root ancestor]] — index 0 is the focused leaf, last index is the root; this ordering drives capture and bubble traversal direction
- [[focus manager propagates key events in capture then bubble phase]] — capture runs root-to-leaf (onCaptureKey), bubble runs leaf-to-root (onKey); returning true from any handler stops propagation
- [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] — the programmatic focus-change API; manages state keys, fires onFocus/onBlur/onFocusChanged on all affected ancestors
- [[focus state key is automatically added and removed from elm states during focus path changes]] — Config.focusStateKey drives the :focus CSS state variant; shared ancestors skip redundant re-adds
- [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]] — the entry point; attaches document keydown/keyup, applies key maps, returns cleanup() and focusPath()
- [[default key map translates browser keys to framework event names]] — maps ArrowLeft→Left, Enter→Enter, Backspace→Back, Escape→Escape, etc.; event names construct handler property names like onLeft, onCaptureEnter
- [[activeElement is a module-level SolidJS signal tracking the currently focused ElementNode]] — the Solid reactive surface for focus changes; injected into Config.setActiveElement to avoid circular imports

## Patterns

- [[element key handlers receive event keyboard-event element and finalFocusElm as arguments]] — capture handlers get (e, elm, finalFocusElm, mappedEvent); bubble handlers get (e, elm, finalFocusElm); fallback handlers (onKeyPress/onKeyHold) swap mappedEvent to second position
- [[ownerContext option in useFocusManager wraps key handlers for reactive framework compatibility]] — use runWithOwner in Solid.js to prevent unowned reactive computations inside key handlers
- [[custom key maps support array values for multiple browser keys and null to delete mappings]] — array values map multiple physical keys to one event; null removes a mapping; all changes are global and permanent for the session
- [[key hold detection fires after 500ms and cancels if key is released early]] — opt-in via userKeyHoldMap; early release cancels hold and fires normal keypress instead; default threshold 500ms

## Gotchas

- [[forwardFocus is required on containers that hold focusable children]] — without forwardFocus, a container becomes a focus dead-end; children never receive focus; no error is thrown
- [[throttleInput can be set globally on Config or per-element to suppress rapid key repeats]] — global throttle drops events entirely (returns false); per-element throttle consumes events (returns true), stopping bubbling; per-element applies to both keydown and keyup
- [[custom key maps support array values for multiple browser keys and null to delete mappings]] — the userKeyMap format is {browserKey: eventName}, but flattenKeyMap inverts the pair internally — easy to misread the direction

## Debug

- [[focus debug mode visualizes the focus path with red borders via data-focus attributes]] — Config.focusDebug=true injects CSS and sets data-focus=1/2/3 attributes; purely cosmetic, no effect on propagation

## Open Questions
- What is Config.focusStateKey's default value?
- How does the states system consume the focus state key to apply CSS variants?
