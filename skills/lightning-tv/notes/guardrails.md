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
2. **`forwardFocus` on every container** that holds focusable children.
3. **`flexGrow` requires >= 2 children.** Single-child flexGrow is silently ignored.
4. **NEVER use `wrap-reverse`.** It is broken in the new flex engine.
5. **Children in flex containers MUST have explicit `width`/`height`.**
6. **Container auto-sizing is the default** (`flexBoundary='contain'`). Set `'fixed'` to prevent it.
7. **Auto-sizing only works with `justifyContent='flexStart'`.**
8. **`style` is set ONCE and locked.** Use signals on individual props or states for dynamic visuals.
9. **`setFocus()` is async** (microtask). Only the last call in a synchronous block wins.
10. **Key handlers MUST `return true`** to stop event propagation.
11. **Always set `color={0xffffffff}`** when using `src` for images.
12. **`textAlign` requires `contain`.** Without it, text alignment is silently ignored.
13. **Never animate before render.** Use `onCreate` or `onRender` callbacks.
14. **Use `Visible` over `Show`** for Lightning nodes that toggle frequently.
15. **State keys MUST start with `$`** (`$focus`, `$active`, `$disabled`).
16. **Never call DOM APIs.** No `document`, no `addEventListener`, no `querySelector`.
17. **Handler signatures differ**: `onLeft(e, target, elm)` vs `onKeyPress(e, mapped, elm, focused)`.
18. **`flattenStyles` is first-wins**, not last-wins. Earlier array entries have higher priority.
19. **Virtual component children receive Accessors** -- call `item()`, not `item`.
20. **Use `scrollToIndex()`** to change selection, never mutate `selected` directly.
21. **Shader properties (border, shadow, rounded) CANNOT be transitioned.** The shader transition path is broken -- `transition={{ border: ... }}` silently fails. Use states for instant switches or animate alpha/scale on a wrapper instead.
22. **linearGradient and radialGradient have NO transition support.** They use a raw shader accessor with zero animation logic.

## Colors

Colors are `0xRRGGBBAA` numeric format, NOT CSS strings:
```tsx
color={0xff0000ff}   // solid red
color={0xffffffff}   // white (required for images)
color={0x00000000}   // transparent
```

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
