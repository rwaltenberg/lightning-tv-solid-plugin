# src Color Side Effect

> Setting src to a string auto-sets color=0xffffffff only if rendered and no color set. Setting src to non-string sets color=0x00000000. Toggling src has color side-effects.

**Source**: `solidjs-integration.md` | **Severity**: critical

## Detail

The `src` setter has implicit color side-effects:

- Setting `src` to a **string**: auto-sets `color = 0xffffffff` (white) **only if** the node is already rendered AND has no prior color set.
- Setting `src` to a **non-string** (e.g., `null`, `undefined`): sets `color = 0x00000000` (transparent).

### The Render Path Edge Case

During initial rendering, if `src` is provided without an explicit `color` and without a `texture`, the render path sets `color = 0x00000000` (transparent) because `!props.color && !props.src` is checked BEFORE `src` is applied to `this.lng`.

This means: if `src` is set pre-render (via JSX prop) and no explicit `color` is set, the node may render as transparent.

### Toggling src

Toggling `src` on/off will have color side-effects:
- Turn on (set to string): color becomes `0xffffffff`
- Turn off (set to null/undefined): color becomes `0x00000000`

### Safe Pattern

Always explicitly set `color` when using `src` to avoid unexpected transparency:

```tsx
<view src="/image.png" color={0xffffffff} />
```

## Code Example

```tsx
// WRONG -- image may not be visible (transparent on initial render)
<view src="/image.png" />

// CORRECT -- always set color explicitly with src
<view src="/image.png" color={0xffffffff} />

// WRONG -- toggling src will unexpectedly change color
const [showImage, setShowImage] = createSignal(true);
<view src={showImage() ? '/image.png' : null} />
// When showImage() becomes false: color is set to 0x00000000

// CORRECT -- handle color explicitly when toggling src
<view
  src={showImage() ? '/image.png' : null}
  color={showImage() ? 0xffffffff : 0x00000000}
/>
```

## Gotchas

- The auto-white-color only fires post-render AND only when no color is already set. Pre-render JSX with `src` but no `color` renders transparent.
- Setting `src` to `null` clears color to `0x00000000` regardless of what was there before.
- The `color` prop accepts `string | number` -- use `hexColor()` utility for string conversion.

## Related Notes

- [core/node-lifecycle.md] -- render() color default logic ("Default color to transparent if no src/texture")
- [core/element-node-proxy.md] -- src is a non-animating prop (direct write to lng)
