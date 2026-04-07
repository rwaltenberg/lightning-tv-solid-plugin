---
description: Setting the LIGHTNING_DISABLE_SHADERS global to true before module load prevents all shader registration and effect application, useful for low-end or non-WebGL targets
type: api
module: core
created: 2026-04-07
---

# LIGHTNING_DISABLE_SHADERS global flag disables all shader effects at build time

`LIGHTNING_DISABLE_SHADERS` is a global variable that must be set before Lightning modules are imported. The exported `SHADERS_ENABLED` constant is derived from it and cannot change at runtime.

```ts
// config.ts
export const SHADERS_ENABLED = !(
  typeof LIGHTNING_DISABLE_SHADERS === 'boolean' && LIGHTNING_DISABLE_SHADERS
);
```

**Enabling (shaders disabled):**
```ts
// vite.config.ts
define: { LIGHTNING_DISABLE_SHADERS: true }
```

`SHADERS_ENABLED` defaults to `true` (shaders enabled) when the global is not set or is not a boolean.

Every shader registration function in `shaders.ts` guards with:
```ts
if (SHADERS_ENABLED && !(DOM_RENDERING && Config.domRendererEnabled))
```

This means both the shader disable flag and DOM rendering mode can independently prevent shader use. Since [[shaders are silently skipped when DOM renderer is active]], the guard handles both paths.

The `convertToShader` function on [[Config singleton holds all runtime Lightning configuration]] is the runtime extension point — it can still be replaced even when default shaders are disabled.

---

Source: [[core-config]], [[core-shaders]]
Domains:
- [[core guide]]
- [[styling guide]]
