---
description: The Announcer reads announce, title, and announceContext from ElementNodes that newly entered the focus path, speaking them in root-to-leaf order with a configurable debounce
type: architecture
module: primitives
created: 2026-04-07
---

# Announcer singleton announces newly focused elements by diffing the focus path against previous state

The `Announcer` is a module-level singleton in `src/primitives/announcer/announcer.ts`. It is the primary accessibility API for Lightning TV/Solid applications.

## Core Mechanism

On every focus path change, `onFocusChangeCore` runs (after debounce):

1. **Loading guard**: If any element in the path has `loading === true`, defers announcement until all are loaded.
2. **Diff**: `focusDiff = focusPath.filter(elm => !prevFocusPath.includes(elm))` — only elements newly entering focus are announced, not the full path.
3. **Content collection** (reversed to root-to-leaf order):
   - `elm.announce` if set (primary speech)
   - `elm.title` as fallback
   - `elm.announceContext` appended after all announce content
4. **Speech**: Flattens collected `SpeechType` values and calls `Announcer.speak()`.
5. **Timer reset**: After `focusChangeTimeout` ms of no focus changes (default 5 minutes), `prevFocusPath` is cleared — causing full re-announcement on next focus change.

## ElementNode Augmentation

The module augments `ElementNode` with accessibility properties:

```typescript
interface ElementNode {
  announce?: SpeechType;        // primary content to speak
  announceContext?: SpeechType; // contextual info (read after announce)
  title?: SpeechType;           // fallback if announce not set
  loading?: boolean;            // defers announcement while true
}
```

`SpeechType` can be a string, array, function, Promise, or `SpeechSynthesisUtterance` — giving flexible content sourcing.

## Announcement Order

For a focus path `[button, row, page]` (leaf-to-root), the Announcer reverses it to read root context first:
1. `page.announce` (or `page.title`)
2. `row.announce` (or `row.title`)
3. `button.announce` (or `button.title`)
4. Then context: `page.announceContext`, `row.announceContext`, `button.announceContext`

This matches how screen readers typically read page → section → item context.

## Required Initialization

`setupTimers()` MUST be called before any announcements work:

```typescript
Announcer.setupTimers({
  focusDebounce: 400,         // wait 400ms after focus change before speaking
  focusChangeTimeout: 300000, // reset prevFocusPath after 5 min of inactivity
});
```

Without this, `Announcer.onFocusChange` is `undefined` and all speech is silently skipped.

## Integration with `focusPath` Signal

```typescript
// Application wiring (typically in App component):
createEffect(() => {
  Announcer.onFocusChange?.(focusPath());
});
```

Since [[focusPath signal is a module-level reactive export updated on every active element change]], the Announcer reacts to focus changes through the Solid reactive system. That signal is updated on every [[setActiveElement updates focus path and fires onFocus and onBlur callbacks]] call — meaning every focus move, including those from `KeepAliveRoute` focus restoration, triggers an Announcer evaluation.

---

Source: [[primitives-announcer]]
Domains:
- [[accessibility guide]]
