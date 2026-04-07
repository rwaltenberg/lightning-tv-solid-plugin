---
description: In Lightning renderer mode, both a src string signal and a texture object signal run in parallel; texture takes precedence once loaded
type: gotcha
module: primitives
created: 2026-04-07
---

# Image component uses dual signal system of src and texture in Lightning renderer mode

When running with the Lightning renderer (not DOM renderer), `Image` manages two reactive signals simultaneously:
- `src` — a string URL, starts as `placeholder`, updated on success/failure
- `texture` — a Lightning `ImageTexture` object, starts as `null`, set only when pixel data confirms successful load

The `<view>` receives both: `src={src()} texture={texture()}`. When `texture()` is non-null, the renderer uses the texture object rather than re-fetching by URL. This means once an image fully loads, it operates as a pre-loaded texture rather than a URL reference.

**Why this matters**: setting `texture` and `src` simultaneously on a view can produce unexpected results if the src is a fallback URL but the texture is still the originally-loaded one. The Image component guards against this — it only calls `setTexture(srcTexture)` when `resp.data` is truthy after `getTextureData()` resolves.

**The failed-texture trap**: `srcTexture.getTextureData()` resolves even after the `'failed'` event fires. Without the `if (resp.data)` guard, a failed load would still set `texture` to the (empty) texture object, overriding the fallback `src` string. Since [[node lifecycle events fire on loaded failed freed and bounds changes]], the `failed` event fires before `getTextureData` resolves.

---

Source: [[primitives-Image]]
Domains:
- [[components guide]]
