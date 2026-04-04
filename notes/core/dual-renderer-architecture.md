# Dual-Renderer Architecture

> The framework supports two renderer backends: lng.RendererMain (WebGL/Canvas) and DOMRendererMain (HTML divs). Selected at init time.

**Source**: `core-rendering-nodes.md` | **Severity**: important

## Detail

The framework implements a dual-renderer architecture with a shared abstraction layer (`ElementNode`) sitting atop two concrete renderer backends:

1. **Lightning Renderer** (`lng.RendererMain`) -- the primary WebGL/Canvas-based GPU renderer designed for TV/embedded devices.
2. **DOM Renderer** (`DOMRendererMain`) -- an experimental fallback that translates the same node tree into standard HTML `<div>` elements with CSS transforms, gradients, and `<img>` elements.

```
SolidJS Reactive Layer
        |
        v
   ElementNode  (virtual node / abstraction)
        |
        v
  ┌─────┴──────┐
  │             │
  v             v
lng.RendererMain   DOMRendererMain
(WebGL/Canvas)     (HTML/CSS divs)
```

### Selection Mechanism

The active renderer is selected at initialization time via:
- **Compile-time global**: `LIGHTNING_DOM_RENDERING` (boolean)
- **Runtime flag**: `Config.domRendererEnabled` (boolean, default: `false`)

Once chosen, a single module-level `renderer` variable (exported from `lightningInit.ts`) holds the instance for the lifetime of the application.

### DOMRendererMain

Each `DOMNode` creates a `<div>` element. Key behaviors:
- Every property setter calls `updateNodeStyles(this)`, which rebuilds the entire inline `style` attribute string.
- Image sources are lazily loaded: `<img>` elements created inside a background layer div (`divBg`) and only have their `src` attribute set when the node's render state is "in bounds".
- Border/radius effects translated to CSS `box-shadow`, `border-radius`, and `border-*` properties.
- Gradients use CSS `linear-gradient` and `radial-gradient`.
- Color tinting of images uses CSS `mask-image` with `mix-blend-mode: multiply`.
- Animations managed by a custom `AnimationController` using `requestAnimationFrame`.

### DomRendererMainSettings

```ts
interface DomRendererMainSettings {
  appWidth?: number;                // default: 1920
  appHeight?: number;               // default: 1080
  deviceLogicalPixelRatio?: number; // default: 1
  boundsMargin?: number | [number, number, number, number];
}
```

### Shaders in DOM Mode

The `registerDefaultShader*` functions explicitly no-op when DOM rendering is active (`!(DOM_RENDERING && Config.domRendererEnabled)`). While the DOM renderer translates some shader effects (border radius, shadows, gradients) to CSS equivalents, complex shader compositions may not render correctly or at all in DOM mode. `createShader` exists on `DOMRendererMain` but its behavior is limited.

### Compile-Time Derived Constants

```ts
export const isDev: boolean;          // true when import.meta.env.DEV is truthy
export const DOM_RENDERING: boolean;  // true when LIGHTNING_DOM_RENDERING === true
export const SHADERS_ENABLED: boolean; // true unless LIGHTNING_DISABLE_SHADERS === true
```

## Code Example

```ts
// DOM renderer mode (for development/debugging without GPU)
// In vite.config.ts:
define: {
  LIGHTNING_DOM_RENDERING: true
}

// Or at runtime:
Config.domRendererEnabled = true;

// Check which mode is active:
import { DOM_RENDERING } from '@lightningtv/solid';
if (DOM_RENDERING) {
  // Skip shader registration
} else {
  registerDefaultShaders(renderer.stage.shManager);
}
```

## Gotchas

- Shaders are no-ops in DOM mode -- do not call `registerDefaultShaders` when `DOM_RENDERING === true`.
- `LIGHTNING_DOM_RENDERING` is a compile-time global (Vite `define`); `Config.domRendererEnabled` is runtime. Both must be considered.
- The renderer is selected once at init -- it cannot be switched at runtime.
- DOM renderer uses `requestAnimationFrame` for animations, not the GPU animation pipeline.

## Related Notes

- [core/rendering-pipeline.md] -- where renderer selection fits in the pipeline
- [api/config.md] -- Config.domRendererEnabled and compile-time globals
