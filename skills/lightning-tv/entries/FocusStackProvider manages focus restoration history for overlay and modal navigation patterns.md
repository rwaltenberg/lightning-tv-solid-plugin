---
description: FocusStackProvider wraps the app or subtree with a Context that exposes storeFocus, restoreFocus, and clearFocusStack for returning focus to a previous element when overlays close
type: api
module: primitives
created: 2026-04-07
---

# FocusStackProvider manages focus restoration history for overlay and modal navigation patterns

`src/primitives/createFocusStack.tsx` provides a focus history stack via Solid.js Context. The pattern is common in TV UIs where focus moves into an overlay, dialog, or submenu and must return to the originating element when the user closes it.

## Setup

```tsx
// App.tsx — wrap the application or relevant subtree
<FocusStackProvider>
  <App />
</FocusStackProvider>
```

## API

```typescript
const { storeFocus, restoreFocus, clearFocusStack } = useFocusStack();
```

**`storeFocus(element, prevElement?)`** — Pushes an element onto the stack. If `prevElement` is provided, it is stored instead of `element`. The `prevElement` parameter names the "return-to" element — useful when the triggering element is not where you want focus to return:

```typescript
// Store the row's selected item as the return target, not the button that opened the modal
storeFocus(currentElement, rowSelectedItem);
```

**`restoreFocus(): boolean`** — Pops the top element and calls `setFocus()` on it. Returns `true` if focus was restored, `false` if the stack was empty.

**`clearFocusStack()`** — Empties the entire stack.

## `useFocusStack(autoClear = true)`

The `autoClear` parameter (default `true`) registers a cleanup that clears the stack with a 5ms delay when the consuming component unmounts:

```typescript
s.onCleanup(() => {
  setTimeout(() => context.clearFocusStack(), 5);
});
```

The 5ms delay is intentional: `restoreFocus()` is typically called in component cleanup, and it must run before the stack is cleared.

## Usage Pattern

```typescript
// In a page component that can open overlays:
const { storeFocus, restoreFocus } = useFocusStack();

function openModal(currentElement: ElementNode) {
  storeFocus(currentElement);
  setModalOpen(true);
  // focus moves to modal...
}

function closeModal() {
  restoreFocus(); // returns to stored element
  setModalOpen(false);
}
```

## Gotchas

- Must be called inside `FocusStackProvider` — throws `"useFocusStack must be used within a FocusStackProvider"` otherwise.
- The 5ms `autoClear` window: if component cleanup takes longer than 5ms before calling `restoreFocus()`, the stack will be empty. In practice this is not an issue since `restoreFocus()` is called synchronously on user action, not during cleanup.
- `storeFocus` is additive — calling it multiple times without `restoreFocus` builds a longer stack. Each `restoreFocus()` call removes one entry.

Since [[setFocus defers focus assignment via microtask to allow children to render first]], `restoreFocus` calling `setFocus()` is safe to call before the target has re-rendered. The actual focus change fires through [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]], triggering `onFocus`/`onBlur` on all affected elements in the path.

For route-level focus restoration (navigating away then back), [[KeepAliveRoute restores focused element when returning to a preserved route]] handles this automatically via router hooks — `FocusStackProvider` is the right choice for transient in-page overlays and modals that don't change the URL.

---

Related Entries:
- [[focus path is an ordered array from focused leaf to root ancestor]] — the path that gets rebuilt when restoreFocus fires setFocus on the stored element

Source: [[primitives-createFocusStack]]
Domains:
- [[focus guide]]
- [[input guide]]
- [[routing guide]]
