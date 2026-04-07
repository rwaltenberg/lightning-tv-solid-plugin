---
description: Creates a test canvas and attempts getContext() for each WebGL ID; results are cached for the default ID list; supportsWebGL/supportsWebGL2/supportsOnlyWebGL2 are predicate wrappers
type: api
module: utils
created: 2026-04-07
---

# getWebglSupportedVersions detects available WebGL context versions by probing canvas contexts

`getWebglSupportedVersions()` determines which WebGL context versions the browser supports by attempting to create rendering contexts:

```ts
const WEBGL_CONTEXT_IDS = ['webgl2', 'webgl', 'experimental-webgl2', 'experimental-webgl'];

export function getWebglSupportedVersions(webglContextIds = WEBGL_CONTEXT_IDS): string[] {
  // Returns cached result if using default list and already computed
  const cv = document.createElement('canvas');
  const supports = webglContextIds.filter((id) => {
    try {
      const context = cv.getContext(id);
      return !!(context && (context instanceof WebGLRenderingContext ||
        context instanceof WebGL2RenderingContext ||
        ('getParameter' in context && ...)));
    } catch { return false; }
  });
  if (webglContextIds === WEBGL_CONTEXT_IDS) {
    supportedWebglVersions = supports; // cache
  }
  return supports;
}
```

Results are cached when using the default `WEBGL_CONTEXT_IDS` array (reference equality check). Custom arrays bypass the cache.

Three predicate wrappers:
- `supportsWebGL(versions)` — any of webgl/experimental-webgl/webgl2 present
- `supportsWebGL2(versions)` — webgl2 specifically present
- `supportsOnlyWebGL2(versions)` — webgl2 present AND webgl/experimental-webgl absent

These are used for renderer initialization decisions — when only WebGL2 is supported (no legacy WebGL), the renderer can take WebGL2-specific optimizations. The decision logic in `startLightningRenderer` uses these predicates to configure the renderer settings appropriately.

---

Source: [[utils]]
Domains:
- [[core guide]]
