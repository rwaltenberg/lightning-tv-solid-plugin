---
description: Setting Config.focusDebug injects a stylesheet and sets data-focus=1/2/3 on focused elements, rendering increasingly faint red borders from leaf to ancestor for visual debugging
type: pattern
module: core
created: 2026-04-07
---

# focus debug mode visualizes the focus path with red borders via data-focus attributes

When `Config.focusDebug` is truthy, the focus manager activates a visual debug overlay that marks every element in the focus path with a `data-focus` attribute.

## How it works

On each focus path update, the manager:
1. Injects a `<style>` block into `document.head` (only once per page load via `needFocusDebugStyles` flag).
2. Removes `data.focus` from all elements that were in the previous focus path.
3. Sets `elm.data = { ...elm.data, focus: i + 1 }` for each element in the new path, where `i=0` is the leaf (focused element) and higher indices are ancestors.

## CSS styles injected

| Attribute       | Appearance                             |
|-----------------|----------------------------------------|
| `data-focus="1"` | 4px solid red border, 90% opacity — the focused element |
| `data-focus="2"` | 2px solid red border, 40% opacity — direct parent |
| `data-focus="3"` | 2px solid red border, 20% opacity — grandparent and beyond |

All borders use `border-radius: 5px` and CSS `transition` for smooth updates.

## Activation

```typescript
// In your app config before rendering
Config.focusDebug = true;
```

Because [[focus path is an ordered array from focused leaf to root ancestor]], `data-focus="1"` is always the currently focused leaf element, making it easy to visually confirm which element has focus at any depth.

This feature is purely cosmetic and has no effect on event propagation or application logic.

---
Source: [[core-focusManager]]
Domains:
- [[focus guide]]
