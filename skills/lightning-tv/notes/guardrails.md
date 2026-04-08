# Lightning TV Guardrails & Quick Reference

## PRIME DIRECTIVE: THIS IS NOT THE WEB DOM

`ElementNode` extends `Object`, **NOT** `HTMLElement`. There is no `document`, no `window.getComputedStyle`, no CSS cascade, no DOM tree. Every `<view>` becomes a `renderer.createNode()` GPU call. Every `<text>` becomes a `renderer.createTextNode()` call. There is no HTML anywhere.

### Developer Mindset

- **Scene graph nodes**, not DOM elements. No `document`, `querySelector`, `classList`, `addEventListener`, `style.cssText`, `innerHTML`.
- **GPU textures and WebGL draw calls**, not CSS paint. Node creation = GPU resource allocation. Minimize creation/destruction.
- **D-pad navigation** (TV remote: arrows, Enter, Back). No mouse cursor, no hover, no scroll wheel by default.
- **Explicit pixel dimensions** required. No intrinsic/auto-sizing. Flex children MUST have `width`/`height`.
- **SolidJS fine-grained reactivity**. Updates write directly to GPU nodes via property proxies.
- **Single-pass synchronous layout**. The flex engine is NOT CSS Flexbox.

## FORBIDDEN CSS LAYOUT MODES

- `display: 'grid'` does NOT exist. Only `display: 'flex'` and `display: 'block'` are supported.

## FORBIDDEN HTML TAGS

These DO NOT EXIST and will cause runtime errors:

```
<div>, <span>, <p>, <h1>-<h6>, <ul>, <ol>, <li>, <a>, <button>,
<input>, <form>, <img>, <section>, <header>, <footer>, <nav>,
<main>, <article>, <aside>, <table>, <canvas>, <video>, <audio>,
<iframe>, <label>, <select>, <textarea>, <details>, <summary>,
<dialog>, <svg>, <picture>, <figure>, <figcaption>
```

### The ONLY Intrinsic JSX Elements

| Tag      | Purpose                          | Maps To                     |
|----------|----------------------------------|-----------------------------|
| `<view>` | Visual rectangle node            | `renderer.createNode()`     |
| `<node>` | Alias for `<view>`               | `renderer.createNode()`     |
| `<text>` | Text rendering node              | `renderer.createTextNode()` |

## FORBIDDEN DOM APIs

- `document.*`, `element.addEventListener()`, `element.classList`, `element.className`
- `element.style.cssText`, `element.innerHTML`, `element.textContent`, `element.parentElement`
- `element.querySelector()`, `window.getComputedStyle()`, `element.getBoundingClientRect()`
- CSS selectors, CSS custom properties, CSS animations, CSS media queries
- `Portal` from `solid-js/web` (use `Portal` from `@lightningtv/solid/primitives`)
- `Suspense` from `solid-js` (use `Suspense` from `@lightningtv/solid/primitives`)

### SolidJS Primitives That Work

`For`, `Show`, `Switch`, `Match`, `Index`, `ErrorBoundary`, `createSignal`, `createMemo`, `createEffect`, `createResource`, `onMount`, `onCleanup`, `batch`, `untrack`

### SolidJS Primitives That DO NOT Work

`Portal` (solid-js/web), `Suspense` (solid-js), `className`, `classList`, `innerHTML`, DOM directives

## NON-NEGOTIABLE RULES

1. **ONLY `<view>`, `<node>`, and `<text>` exist.** No HTML tags. Ever.
2. **`forwardFocus` on every container** that holds focusable children. Without it, the container absorbs focus and children's handlers never fire.
3. **`flexGrow` requires >= 2 children.** The flex engine guards with `numProcessedChildren > 1`; single-child flexGrow is silently ignored.
4. **Children in flex containers MUST have explicit `width`/`height`.** GPU nodes don't auto-size from content.
5. **Container auto-sizing is the default** (`flexBoundary='contain'`). Set `'fixed'` to prevent it.
6. **Auto-sizing only works with `justifyContent='flexStart'`.**
7. **`style` is write-once initial state, not reactive.** Use it for static base properties and `$`-prefixed state variants (`$focus`, `$active`, `$disabled`). JSX inline props override `style` — the setter skips any key already set on the element. For dynamic/reactive values, use JSX inline props with signals. Re-setting `style` after mount warns in dev mode and is blocked by `Config.lockStyles`.
8. **`setFocus()` is async** (microtask). Only the last call in a synchronous block wins.
9. **Key handlers MUST `return true`** to stop event propagation.
10. **Always set `color={0xffffffff}`** when using `src` for images. Use `color={0x00000001}` (near-transparent) for nodes that will receive `src` later dynamically.
11. **`textAlign` requires `contain`.** Without it, text alignment is silently ignored.
12. **Never animate before render.** Use `onCreate` or `onRender` callbacks.
13. **Use `Visible` over `Show`** for Lightning nodes that toggle frequently.
14. **State keys MUST start with `$`** (`$focus`, `$active`, `$disabled`).
15. **Never call DOM APIs.** No `document`, no `addEventListener`, no `querySelector`.
16. **Handler signatures differ by phase:**
    - Bubble specific: `onLeft(e, elm, finalFocusElm)` — returns `boolean | void`
    - Bubble fallback: `onKeyPress(e, mappedEvent, elm, finalFocusElm)` — note `mappedEvent` is second
    - Capture: `onCaptureLeft(e, elm, finalFocusElm, mappedEvent?)` — `mappedEvent` is last
17. **`flattenStyles` is first-wins**, not last-wins. Earlier array entries have higher priority.
18. **Virtual component children receive Accessors** -- call `item()`, not `item`.
19. **Use `scrollToIndex()`** to change selection on Row/Column/Virtual. Don't bypass it by setting `selected` externally on Virtual without also scrolling.
20. **`selectedNode` getter is a side effect** -- reading it scans forward and mutates `this.selected`. Avoid in `createEffect`.
21. **`linearGradient`, `radialGradient`, and `border` have NO transition support.** Gradients use a raw shader accessor with zero animation logic. Border transitions silently fail because the `shaderAccessor` writes border props (`border-color`, `border-w`, etc.) into a detached object and passes it to `animate({ shaderProps })`, but the renderer cannot interpolate compound shader keys. `borderRadius` transitions DO work (single numeric `radius` value).
22. **`wrap-reverse` may produce unexpected results.** The reversal interacts with RTL direction; combining both produces a single reversal, not a double. **Critical:** In the new flex engine (`VITE_USE_NEW_FLEX`), the wrapping condition only checks `flexWrap === 'wrap'` — `wrap-reverse` is excluded from the wrapping logic entirely, making it effectively broken.
23. **`VITE_USE_NEW_FLEX` changes flex behavior.** The new engine adds `flexShrink`, `flexBasis`, `flexCrossBoundary`, and per-side padding (`paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft`). But it also changes cross-alignment to account for padding (shifting baseline alignment vs the old engine), removes the `console.warn` when flex-grow has no available space, and breaks `wrap-reverse` (see rule 22). Check which engine your project uses — props like `flexShrink` and individual padding silently no-op on the old engine.

## Positioning Shortcuts

`right` and `bottom` props automatically set `mountX: 1` and `mountY: 1` respectively. `right={0}` pins the node's right edge flush with the parent's right edge. These are computed once during `render()` — they do NOT reactively update if the parent resizes.

`center`, `centerX`, `centerY` similarly set `mount` to `0.5` and offset by half the parent dimension.

## Colors

Colors are `0xRRGGBBAA` numeric format. String formats (`"#RRGGBB"`, `"#RRGGBBAA"`) also work via `hexColor()` conversion, but the codebase convention is numeric:
```tsx
color={0xff0000ff}   // solid red
color={0xffffffff}   // white (required for images)
color={0x00000000}   // transparent
color={0x00000001}   // near-transparent (safe default for dynamic src nodes)
```
Named CSS colors (e.g., `'red'`, `'blue'`) are NOT supported.

## Application Bootstrap

```tsx
import { createRenderer, Config } from '@lightningtv/solid';
import { useFocusManager } from '@lightningtv/solid/primitives';

Config.rendererOptions = { appWidth: 1920, appHeight: 1080 };
const { render } = createRenderer(undefined, 'app');

const App = () => {
  useFocusManager();  // ONCE, at the root
  return (
    <view width={1920} height={1080} color={0x000000ff}>
      {/* Your application */}
    </view>
  );
};

render(() => <App />);
```

## Component Anatomy

```tsx
import { type NodeStyles } from '@lightningtv/solid';

const cardStyle: NodeStyles = {
  width: 300, height: 200, color: 0x1a1a1aff, borderRadius: 12,
  $focus: { color: 0x333333ff, scale: 1.05 },
};

const Card = (props) => (
  <view style={cardStyle} onEnter={() => { props.onSelect?.(); return true; }}>
    <text y={20} x={20} fontSize={28} color={0xffffffff}>{props.title}</text>
  </view>
);
```
