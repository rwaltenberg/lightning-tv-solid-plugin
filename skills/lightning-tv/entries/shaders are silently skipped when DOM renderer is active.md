---
description: All registerDefaultShader* functions no-op when DOM rendering is enabled — shader effects including rounded corners, shadows, and gradients are unavailable in DOM mode
type: gotcha
module: core
created: 2026-04-07
---

# shaders are silently skipped when DOM renderer is active

Every shader registration function in `shaders.ts` is guarded by the same condition:

```ts
if (SHADERS_ENABLED && !(DOM_RENDERING && Config.domRendererEnabled)) {
  shManager.registerShaderType(key, shader);
}
```

When the DOM renderer is active, this guard evaluates to `false` and no shader is registered. There is no error or warning — shader properties simply have no effect.

**What this means in practice:**
- `borderRadius`, rounded corners, drop shadows, radial/linear gradients — all implemented as WebGL shaders — produce no visual output in DOM mode
- `registerDefaultShaders()` (batch registration) also silently skips all shaders
- The DOM renderer is primarily used for testing or SSR; visual parity with WebGL is not guaranteed

**Workaround for DOM mode:** Use CSS-based styling on the host elements if visual correctness in DOM mode is required.

Since [[dom renderer requires both build-time flag and runtime Config flag to activate]], DOM mode must be explicitly opted into — shaders work by default in a standard WebGL setup.

Since [[LIGHTNING_DISABLE_SHADERS global flag disables all shader effects at build time]], the `SHADERS_ENABLED` part of the guard can independently disable shaders even in WebGL mode.

---

Source: [[core-shaders]]
Domains:
- [[core guide]]
- [[styling guide]]
