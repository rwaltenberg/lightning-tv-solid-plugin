---
description: Accepts src, placeholder (shown during load), and fallback (shown on error); both DOM and Lightning renderer paths are supported
type: api
module: primitives
created: 2026-04-07
---

# Image component shows placeholder while src loads and fallback on error

The `Image` component extends `NodeProps` with three image-specific props:

```ts
interface ImageProps extends NodeProps {
  src: string;
  placeholder?: string;  // url shown during loading
  fallback?: string;     // url shown if src fails
}
```

On mount, `src` signal is initialized to `placeholder` (or `null`). The real `src` is loaded in the background; the signal is updated to the real URL only after successful load. If loading fails and `fallback` is set (and `fallback !== placeholder`), the signal updates to `fallback` instead.

The rendered element defaults `color` to `0xffffffff` (white) if no color prop is provided.

```tsx
<Image
  src="https://cdn.example.com/poster.jpg"
  placeholder="https://cdn.example.com/poster-thumb.jpg"
  fallback="https://cdn.example.com/default.jpg"
  width={300}
  height={180}
/>
```

Since [[dom renderer requires both build-time flag and runtime Config flag to activate]], the Image component uses two separate code paths — a `window.Image` preload approach for DOM and a Lightning `ImageTexture` approach for the renderer.

Since [[Lazy delay prop debounces item loading during rapid navigation]], wrapping Image in a LazyRow with `delay` avoids triggering network requests for images the user scrolls past quickly — the image only starts loading when the user pauses on an item.

Since [[createBlurredImage reactive hook produces blurred data URLs using CPU-side Gaussian convolution]], the placeholder pattern pairs naturally: pass `blurred()` as `placeholder` to show a blurred preview while the full image loads.

---

Source: [[primitives-Image]]
Domains:
- [[components guide]]
