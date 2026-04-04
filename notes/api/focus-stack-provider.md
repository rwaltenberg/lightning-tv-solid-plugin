# FocusStackProvider API

> FocusStackContextType interface: storeFocus, restoreFocus, clearFocusStack. autoClear defaults to true with 5ms setTimeout delay on clearFocusStack.

**Source**: `spatial-navigation.md` | **Severity**: informational

## Detail

Located in `src/primitives/createFocusStack.tsx`.

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

**`storeFocus(element, prevElement?)`**: Pushes the current focus element (and optionally a previous element) onto the stack. Call this before navigating to a modal or overlay.

**`restoreFocus(): boolean`**: Pops the stack and restores focus to the previously stored element. Returns `boolean` indicating success.

**`clearFocusStack()`**: Empties the entire stack.

**`autoClear` parameter on `useFocusStack`**: Defaults to `true`. When `true`, registers an `onCleanup` callback that calls `clearFocusStack()` via a 5ms `setTimeout` after the component unmounts. The 5ms delay is intentional — it allows exit animations to still trigger `restoreFocus()` during their cleanup. If `restoreFocus()` is not called before the 5ms window closes, the stored focus is permanently lost.

**`useFocusStack()` throws** if called without a `FocusStackProvider` ancestor in the component tree.

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

- `useFocusStack()` throws an error if there is no `FocusStackProvider` ancestor.
- With `autoClear = true`, stored focus is cleared 5ms after the component unmounts. Always call `restoreFocus()` synchronously during cleanup.
- The 5ms delay in `autoClear` is a `setTimeout`, not a SolidJS reactive primitive — it runs outside the reactive graph.

## Related Notes

- [focus/focus-stack.md] -- usage guide for the focus stack in modal flows
