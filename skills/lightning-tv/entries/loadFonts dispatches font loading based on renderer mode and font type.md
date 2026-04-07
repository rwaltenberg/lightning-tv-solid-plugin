---
description: loadFonts(fonts) routes each font to the correct loading API based on renderer mode (webgl vs canvas) and font format (msdf/ssdf vs fontUrl) — mismatched fonts are silently skipped
type: api
module: core
created: 2026-04-07
---

# loadFonts dispatches font loading based on renderer mode and font type

`loadFonts(fonts: FontLoadOptions[])` handles font registration for both WebGL and canvas/DOM rendering modes. Font loading must happen AFTER `startLightningRenderer`.

```ts
import { startLightningRenderer, loadFonts } from '@lightning/solid';

startLightningRenderer(options);

loadFonts([
  // SDF font for WebGL mode
  {
    type: 'msdf',
    fontFamily: 'Roboto',
    atlasUrl: '/fonts/Roboto.png',
    atlasDataUrl: '/fonts/Roboto.json',
  },
  // Web font for canvas/DOM mode
  {
    fontFamily: 'Roboto',
    fontUrl: '/fonts/Roboto.woff2',
    descriptors: { weight: '400' },
  },
]);
```

**Dispatch logic:**
1. If `renderer.stage.renderer.mode === 'webgl'` AND font has `type: 'msdf' | 'ssdf'` → `renderer.stage.loadFont('sdf', font)`
2. If font has `fontUrl` AND DOM renderer active → `loadFontToDom(font)`
3. If font has `fontUrl` AND NOT webgl mode → `renderer.stage.loadFont('canvas', font)`
4. Otherwise → **silently skipped, no warning**

**Gotcha:** An SDF font with `type: 'msdf'` will be silently ignored in canvas mode. A web font with `fontUrl` will be silently ignored in WebGL mode. If fonts are not rendering, check that the font format matches the renderer mode.

Since [[startLightningRenderer initializes the global renderer singleton]], the renderer must exist before `loadFonts` is called — `renderer.stage` would otherwise be undefined.

---

Source: [[core-lightningInit]]
Domains:
- [[core guide]]
