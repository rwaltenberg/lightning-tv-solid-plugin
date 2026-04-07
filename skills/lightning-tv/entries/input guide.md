---
description: Input handling — useFocusManager hook, custom key maps, hold detection, mouse events and collision detection
type: moc
---

# input guide

Input handling covers keyboard events (TV remote simulation), mouse support, custom key mapping, and hold detection. The `useFocusManager` hook is the primary entry point for setting up input handling in a Solid.js Lightning TV application.

## Core Concepts

- [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]] — the core input entry point; attaches keydown/keyup listeners to document and drives all event propagation
- [[primitives useFocusManager wraps core focus manager with Solid reactive owner context]] — the Solid.js adapter that wires reactive ownership and exports `focusPath` signal; call this in the app root, not the core version
- [[focusPath signal is a module-level reactive export updated on every active element change]] — module-level `createSignal<ElementNode[]>` accessible anywhere; primary integration point for the Announcer and other reactive consumers
- [[ownerContext option in useFocusManager wraps key handlers for reactive framework compatibility]] — how Solid reactive ownership is maintained inside DOM event listeners via `runWithOwner`

## Key Handling & Hold Detection

- [[default key map translates browser keys to framework event names]] — maps `ArrowLeft → Left`, `Enter → Enter`, `Backspace → Back`, etc.; can be overridden or extended
- [[custom key maps support array values for multiple browser keys and null to delete mappings]] — how to customize key mapping at the framework level
- [[key hold detection fires after 500ms and cancels if key is released early]] — global hold detection for keys in the hold map (e.g., `EnterHold`)
- [[useHold distinguishes quick press from long hold using a threshold timeout and wasHeld flag]] — per-component hold primitive returning `[startHold, releaseHold]` handlers; use for directional hold (Right hold, Left hold, etc.)
- [[DefaultKeyHoldMap adds hold-key detection for EnterHold]] — the default hold key map
- [[KeyHandlerReturn true stops key event propagation up the focus chain]] — how to consume an event without further propagation

## Event Propagation

- [[focus manager propagates key events in capture then bubble phase]] — root-to-leaf capture, then leaf-to-root bubble; handlers can stop propagation by returning `true`
- [[element key handlers receive event keyboard-event element and finalFocusElm as arguments]] — the handler signature for all `on${Event}` callbacks

## Mouse Support

- [[useMouse adds pointer support via BFS hit testing over the Lightning element tree]] — BFS hit testing using `absX`/`absY` + DPR scaling; hover tracking, click handling, scroll translation
- [[useMouse custom states mode separates hover and pressed visual feedback from focus management]] — use `customStates: { hoverState, pressedState }` when hover should not change keyboard focus
- [[useMouse click falls back to synthetic Enter keyboard event when element has no explicit click handler]] — click → `onMouseClick` → `onEnter` → synthetic `Enter` keyboard event; no special mouse handler needed for standard actions

## Focus Stack (Overlay/Modal Navigation)

- [[FocusStackProvider manages focus restoration history for overlay and modal navigation patterns]] — Context-based stack with `storeFocus`, `restoreFocus`, and `clearFocusStack`; wrap the app or relevant subtree with `<FocusStackProvider>`

## Input Throttling

- [[throttleInput can be set globally on Config or per-element to suppress rapid key repeats]] — per-element or global throttle to prevent rapid key repeat from flooding handlers

## Patterns

**Setting up input for a Solid.js Lightning TV app:**
```typescript
// In App.tsx root component
useFocusManager(); // primitives version — wires Solid reactivity

// With custom key map:
useFocusManager({ myKey: 'CustomEvent' });
```

**Per-direction hold detection:**
```typescript
const [holdRight, releaseRight] = useHold({
  onHold: handleScrollRight,
  onEnter: handleSelectRight,
  holdThreshold: 200,
});
<View onRight={holdRight} onRightRelease={releaseRight} />
```

**Adding mouse support:**
```typescript
useMouse(); // default: hover changes focus
// or with visual states:
useMouse(rootNode, 100, { customStates: { hoverState: '$hover', pressedState: '$pressed' }});
```

**Focus stack for modal navigation:**
```tsx
<FocusStackProvider>
  <App />
</FocusStackProvider>
// In component:
const { storeFocus, restoreFocus } = useFocusStack();
storeFocus(currentElement); // before opening modal
restoreFocus();             // when modal closes
```

## Gotchas

- **Import `useFocusManager` from primitives, not core**: The core version does not set up Solid.js reactive signals or owner context. Always import from `@lightningtv/solid` or `src/primitives/useFocusManager.js`.
- **Call `useFocusManager` once**: Calling it multiple times attaches duplicate DOM listeners.
- **`useHold` vs global hold map**: Global hold map (`keyHoldOptions.userKeyHoldMap`) affects all elements. `useHold` is per-component and handles specific directions independently.
- **`useMouse` scroll translation**: Wheel events become `ArrowUp`/`ArrowDown` keyboard events — the key hold system will also receive these.

## Open Questions

- How does `throttleInput` interact with `useHold`? Can throttle interfere with hold detection timing?
- Is there a way to disable scroll-to-key translation in `useMouse` while keeping hover?
