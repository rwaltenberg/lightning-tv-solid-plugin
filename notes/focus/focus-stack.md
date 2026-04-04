# Focus Stack for Modals and Overlays

> FocusStackProvider + useFocusStack() manage focus state during route transitions or overlay/modal flows. autoClear clears the stack 5ms after unmount.

**Source**: `spatial-navigation.md` | **Severity**: important

## Detail

The focus stack (`src/primitives/createFocusStack.tsx`) is a SolidJS Context-based stack for storing and restoring focus state. It is used for modal/overlay flows where focus needs to return to a previous location after the overlay closes.

API:

```ts
interface FocusStackContextType {
  storeFocus: (element: ElementNode, prevElement?: ElementNode) => void;
  restoreFocus: () => boolean;
  clearFocusStack: () => void;
}

export function FocusStackProvider(props: { children: JSX.Element }): JSX.Element;
export function useFocusStack(autoClear?: boolean): FocusStackContextType;
// autoClear defaults to true
```

`useFocusStack()` throws an error if there is no `FocusStackProvider` ancestor in the component tree.

**`autoClear` behavior:** By default (`autoClear = true`), `useFocusStack` registers an `onCleanup` that calls `clearFocusStack()` with a 5ms `setTimeout` delay. This means after the component unmounts, there is a brief window where `restoreFocus()` can still work (by design, for exit animations). However, if the component unmounts before `restoreFocus()` is called, the stack will be cleared 5ms later and the stored focus is lost.

Always call `restoreFocus()` synchronously during cleanup/exit logic.

## Code Example

```tsx
function MyPage() {
  return (
    <FocusStackProvider>
      <PageContent />
    </FocusStackProvider>
  );
}

function PageContent() {
  const { storeFocus, restoreFocus } = useFocusStack();

  function openModal(currentElm: ElementNode) {
    storeFocus(currentElm);
    // navigate to modal, which takes focus
  }

  function closeModal() {
    restoreFocus(); // returns focus to where it was
  }
}
```

## Gotchas

- `useFocusStack()` throws if called outside a `FocusStackProvider` ancestor.
- With `autoClear = true` (the default), stored focus is cleared 5ms after the component unmounts. Call `restoreFocus()` synchronously during cleanup to avoid losing it.
- The 5ms delay is intentional to support exit animations that may still need to trigger focus restoration.

## Related Notes

- [api/focus-stack-provider.md] -- full API for FocusStackContextType interface
