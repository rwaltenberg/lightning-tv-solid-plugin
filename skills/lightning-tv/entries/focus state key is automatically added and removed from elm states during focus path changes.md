---
description: The focus manager adds Config.focusStateKey to elm.states when an element enters focus and removes it when it leaves; this drives the :focus state variant in the states styling system
type: architecture
module: core
created: 2026-04-07
---

# focus state key is automatically added and removed from elm states during focus path changes

The focus manager is the sole authority over focus-related state keys on elements. During every `setActiveElement` call, [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] runs and manages `Config.focusStateKey` across all affected elements.

## What the state key does

`Config.focusStateKey` (typically `'focus'`) is the string key added to an element's `states` set. Because the states system applies style variants using dollar-prefix naming, having `'focus'` in states activates the `:focus` style variant automatically:

```typescript
// Any element in the focus path gets this:
elm.states.add(Config.focusStateKey);   // on entering focus path
elm.states.remove(Config.focusStateKey); // on leaving focus path
```

## Optimization: shared ancestors are not re-added

The code only calls `states.add` if the element does not already have the focus state key — unless the element is the exact newly focused leaf (`currentFocusedElm`). This prevents redundant state updates and callback fires on ancestors that remained in focus across navigation.

```typescript
if (!current.states.has(Config.focusStateKey) || current === currentFocusedElm) {
  current.states.add(Config.focusStateKey);
  current.onFocus?.call(...);
  current.onFocusChanged?.call(...);
}
```

This means: if a Row container was already in focus when navigating between its children, `onFocus` is NOT called on the Row again — only on the newly focused child and on newly added ancestors.

## Configurable key name

`Config.focusStateKey` is configurable, so apps can use a custom focus state name if needed. All elements in the system use the same key, so changing it affects the entire app.

Since [[dollar-prefix state keys in NodeStyles apply style variants based on active states]], the state key added here (e.g., `'$focus'`) automatically activates any matching `$focus` block in a component's style object. This is the bridge between the focus manager and the visual styling system — navigation events directly drive style variant changes without any application code.

---

Related Entries:
- [[Config.focusStateKey controls which state string marks focused elements]] — the Config field that names the key added here
- [[_stateChanged applies dollar-prefix style variants with undo tracking and state order priority]] — triggered by states.add/remove; this is what actually applies the $focus visual styles

Source: [[core-focusManager]]
Domains:
- [[focus guide]]
- [[styling guide]]
