# RTT Color Defaults and Transparent Color Falsy Trap

> RTT (render-to-texture) nodes default to white (0xffffffff), not transparent. Regular elements without src/texture default to transparent (0x00000000). Color value 0x00000000 is falsy in JavaScript, which causes unexpected auto-overrides.

**Source**: `elementNode.ts` (lines 1060, 1404-1412) | **Severity**: important

## Detail

### RTT vs Regular Color Defaults

During rendering, elements without explicit color get different defaults:

| Condition | Default Color | Visual |
|-----------|--------------|--------|
| RTT node, no color | `0xffffffff` (white) | Opaque white background |
| Regular node, no src/texture | `0x00000000` (transparent) | Invisible |
| Regular node with src | `0xffffffff` (white) | Image visible (if color set post-render) |

### The Transparent Color Falsy Trap

`0x00000000` (transparent) is `0` in JavaScript, which is falsy. Multiple places in the codebase use `!this.color` to check if color is "unset", but this also matches intentionally transparent nodes:

```ts
// src setter (line 1060):
if (!this.color && this.rendered) {
  this.color = 0xffffffff;  // Overwrites intentional transparency!
}
```

If you set `color={0x00000000}` to make a node transparent, then later set `src`, the color is auto-overridden to white because `!0` is `true`.

## Code Example

```tsx
// SURPRISE: RTT node has opaque white background
<view rtt width={200} height={200}>
  <view color={0xff0000ff} width={50} height={50} />
</view>
// Fix: explicitly set color on RTT node
<view rtt width={200} height={200} color={0x00000000}>
  <view color={0xff0000ff} width={50} height={50} />
</view>

// SURPRISE: transparent color gets overridden when src is set later
const [img, setImg] = createSignal<string | undefined>(undefined);
<view color={0x00000000} src={img()} />
// When setImg('/photo.png') is called:
//   !this.color → !0 → true → color auto-set to 0xffffffff
// Fix: use color={0x00000001} (nearly transparent but truthy) or always set color explicitly
```

## Gotchas

- There is no safe way to have a truly transparent node that later receives an `src` without the color being auto-overridden. Use `color={0x00000001}` as a workaround (alpha = 1/255, visually transparent but truthy).
- RTT nodes are used internally by `FadeInOut` for exit animations. The white default is intentional there (it needs a visible snapshot), but surprising if you create RTT nodes for other purposes.

## Related Notes

- [constraints/src-color-side-effect.md] -- full src/color interaction details
- [api/fade-in-out.md] -- uses rtt internally
