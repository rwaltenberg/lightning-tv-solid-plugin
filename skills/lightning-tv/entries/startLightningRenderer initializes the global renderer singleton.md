---
description: startLightningRenderer(options, rootId?) creates either a WebGL RendererMain or DOMRendererMain and assigns it to the module-level renderer singleton
type: api
module: core
created: 2026-04-07
---

# startLightningRenderer initializes the global renderer singleton

`startLightningRenderer` is the primary initialization function. It must be called before creating any nodes.

```ts
import { startLightningRenderer } from '@lightning/solid';

const renderer = startLightningRenderer(
  {
    appWidth: 1920,
    appHeight: 1080,
    deviceLogicalPixelRatio: 1,
  },
  'app', // HTML element id or HTMLElement — defaults to 'app'
);
```

The function:
1. Checks `DOM_RENDERING && Config.domRendererEnabled` to choose backend
2. Creates `DOMRendererMain` or `lng.RendererMain` accordingly
3. Assigns to the module-level `renderer` export (accessible via `getRenderer()`)
4. Returns the renderer instance

**Calling it twice replaces the renderer singleton.** There is no cleanup of the previous renderer instance — avoid re-calling in production code.

Since [[dom renderer requires both build-time flag and runtime Config flag to activate]], both must be set for DOM mode to take effect.

Since [[loadFonts dispatches font loading based on renderer mode and font type]], font loading should happen after `startLightningRenderer` because it inspects `renderer.stage.renderer.mode`.

---

Source: [[core-lightningInit]]
Domains:
- [[core guide]]
