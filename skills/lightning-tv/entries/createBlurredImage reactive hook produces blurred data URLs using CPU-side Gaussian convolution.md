---
description: Returns a SolidJS Resource that reactively re-blurs when imageUrl changes; uses CSS filter when available, falls back to separable Gaussian convolution
type: api
module: primitives
created: 2026-04-07
---

# createBlurredImage reactive hook produces blurred data URLs using CPU-side Gaussian convolution

`createBlurredImage` is a SolidJS hook that blurs an image reactively and returns the result as a data URL suitable for Lightning texture props.

```ts
import { createBlurredImage } from '@lightningtv/solid/primitives';

const imageUrl = () => props.src;
const blurred = createBlurredImage(imageUrl, { radius: 15, resolution: 0.5 });

// In JSX:
<view src={blurred()} />
<view hidden={blurred.loading} />  // Resource has .loading accessor
```

**Options**:
```ts
{
  radius?: number;       // blur radius in pixels (default 10, must be > 0)
  crossOrigin?: 'anonymous' | 'use-credentials' | '';  // default 'anonymous'
  resolution?: number;   // scale factor 0-1 (default 1); lower = smaller canvas = faster
}
```

**Reactive behavior**: the underlying `createResource` re-runs `applyGaussianBlur` whenever `imageUrl()` changes. Returns `null` when `imageUrl()` is `null` or `undefined`.

**Blur algorithm**: uses `canvas.getContext('2d').filter = 'blur(Npx)'` when available (hardware-accelerated in most browsers). Falls back to a two-pass separable Gaussian convolution (horizontal then vertical pass) using `ImageData` pixel manipulation.

**`resolution` parameter**: scales the canvas before blurring. `resolution: 0.5` produces a quarter-size canvas, reducing both memory usage and computation time. Since blur is meant to obscure detail, lower resolution is often acceptable.

**`applyGaussianBlur(imageUrl, options): Promise<string>`** is also exported standalone for imperative use.

This is a pure CPU/canvas operation with no Lightning renderer involvement, making it compatible with both DOM and Lightning renderer modes. Since [[Image component shows placeholder while src loads and fallback on error]], the output data URL can be passed as `placeholder` to show a blurred preview while the full image loads.

---

Source: [[primitives-utils-createBlurredImage]]
Domains:
- [[components guide]]
