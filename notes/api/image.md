# Image

> Lightning-aware image component with placeholder, fallback, and renderer-specific load paths.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`Image` has two code paths depending on the renderer:
- **DOM renderer**: uses `window.Image` for preloading.
- **Lightning renderer**: uses `renderer.createTexture('ImageTexture', ...)`.

Shows `placeholder` immediately if provided, then swaps to `src` on successful load. On failure, falls back to `fallback` unless `fallback === placeholder` (guard against infinite loop). Sets `color` to `0xffffffff` by default (fully opaque white tint).

**Export**: `export const Image: Component<ImageProps>`

## Props / API

```ts
interface ImageProps extends NodeProps {
  src: string;
  placeholder?: string;   // shown while loading
  fallback?: string;       // shown on error
}
```

| Prop | Type | Description |
|------|------|-------------|
| `src` | `string` | URL of the image to load |
| `placeholder` | `string` | Shown immediately while `src` is loading |
| `fallback` | `string` | Shown on load error |

Default `color`: `0xffffffff` (fully opaque white tint).

## Code Example

```tsx
import { Image } from '@lightningtv/solid/primitives';

<Image
  src="https://example.com/poster.jpg"
  placeholder="/assets/placeholder.png"
  fallback="/assets/error.png"
  width={300}
  height={200}
/>
```

## Gotchas

- If `fallback === placeholder`, the fallback is NOT applied (guards against an infinite error loop).
- Default `color` is `0xffffffff`. Override via NodeProps `color` if a different tint is needed.
- Two separate code paths exist for DOM renderer vs Lightning renderer -- the loading mechanism differs.

## Related Notes

- [api/fade-in-out.md] -- can be combined with FadeInOut for animated image transitions
