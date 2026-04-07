---
description: DOM renderer only activates when LIGHTNING_DOM_RENDERING global is true AND Config.domRendererEnabled is true — either alone is insufficient
type: gotcha
module: core
created: 2026-04-07
---

# dom renderer requires both build-time flag and runtime Config flag to activate

The DOM renderer (used for testing and non-WebGL environments) has a two-gate activation condition:

```ts
// In config.ts:
export const DOM_RENDERING =
  typeof LIGHTNING_DOM_RENDERING === 'boolean' && LIGHTNING_DOM_RENDERING;

// In lightningInit.ts:
const enableDomRenderer = DOM_RENDERING && Config.domRendererEnabled;
```

Both must be true for `DOMRendererMain` to be used instead of `lng.RendererMain`.

**Correct setup:**
```ts
// In your bundler/build config (e.g., vite.config.ts):
define: { LIGHTNING_DOM_RENDERING: true }

// In your app initialization:
Config.domRendererEnabled = true;
startLightningRenderer(options);
```

**Gotcha:** Setting only `Config.domRendererEnabled = true` at runtime without the build-time global will silently fall back to the WebGL renderer. There is no warning or error.

The same dual-gate applies to shader registration — since [[shaders are silently skipped when DOM renderer is active]], no shaders are registered in DOM mode regardless of `SHADERS_ENABLED`.

The `SHADERS_ENABLED` flag has a similar build-time mechanic — since [[LIGHTNING_DISABLE_SHADERS global flag disables all shader effects at build time]], it must be set before the module loads.

---

Source: [[core-config]], [[core-lightningInit]]
Domains:
- [[core guide]]
