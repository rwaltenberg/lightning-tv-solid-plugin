---
description: focusPath is a createSignal<ElementNode[]> declared at module scope in primitives/useFocusManager, persists for the app lifetime, and is consumed directly by the Announcer
type: api
module: primitives
created: 2026-04-07
---

# focusPath signal is a module-level reactive export updated on every active element change

`focusPath` is exported from `src/primitives/useFocusManager.ts` as a module-level Solid.js signal:

```typescript
const [focusPath, setFocusPath] = createSignal<ElementNode[]>([]);
export { focusPath };
```

Because it is declared at module scope (not inside any component or hook), it:
- **Persists for the entire application lifetime** — it is created once when the module is first imported.
- **Is globally accessible** — any module that imports it gets the same signal reference.
- **Does not require a component context to read** — can be called from effects, stores, or non-component code.

The signal is updated inside `useFocusManager()`:

```typescript
createEffect(
  on(activeElement, () => {
    setFocusPath([...focusPathCore()]);
  }, { defer: true }),
);
```

A spread copy (`[...focusPathCore()]`) is stored so that signal consumers get a stable array reference that does not change when the core's internal array mutates.

## Primary Consumer: Announcer

The Announcer reads from `focusPath` to trigger speech on focus changes:

```typescript
import { focusPath } from '../useFocusManager.js';
// in Announcer.refresh:
Announcer.onFocusChange(untrack(() => focusPath()));
```

Application code can also consume `focusPath()` directly in effects:

```typescript
createEffect(() => {
  const path = focusPath();
  Announcer.onFocusChange?.(path);
});
```

Since [[primitives useFocusManager wraps core focus manager with Solid reactive owner context]], the signal update runs within the reactive owner, so downstream effects are properly tracked.

The signal fires on every [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] call, since `Config.setActiveElement` is overridden in the primitives layer to call `setActiveElement` within the reactive owner — making `activeElement()` reactive and triggering the effect that refreshes `focusPath`.

---

Source: [[primitives-useFocusManager]]
Domains:
- [[input guide]]
- [[accessibility guide]]
