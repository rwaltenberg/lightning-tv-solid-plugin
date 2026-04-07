---
description: Non-texture element nodes without a src or color get color = 0x00000000 during render(); setting src after render requires manually setting color to 0xffffffff
type: gotcha
module: core
created: 2026-04-07
---

# element nodes default color to transparent unless src or explicit color is set

During `render()`, element nodes (non-text) without a `texture` go through default handling:

```typescript
if (!props.color && !props.src) {
  // Default color to transparent - If you later set a src, you'll need
  // to set color '#ffffffff'
  props.color = 0x00000000;
}
```

This means an element node with no explicit color or src will be invisible (transparent). This is intentional — containers don't need a visible fill by default.

## The src-after-render gotcha

If you set `src` before rendering (via JSX prop), `props.src` is already set when this check runs, so the default transparent is skipped and the image renders correctly.

But if you set `src` dynamically after the node is rendered:

```typescript
// ElementNode src setter:
set src(src) {
  if (typeof src === 'string') {
    this.lng.src = src;
    if (!this.color && this.rendered) {
      this.color = 0xffffffff;  // auto-sets white when setting src post-render
    }
  } else {
    this.color = 0x00000000;  // clear color when src is falsy
  }
}
```

The `src` setter handles the post-render case — if `color` is not already set, it auto-applies `0xffffffff`. But if `color` was explicitly set to `0x00000000` (e.g., because a prior render defaulted it), the check `!this.color` passes and white is applied.

**Safe pattern:** Always set an explicit `color` when using images that may be dynamically loaded or swapped:

```jsx
<View src={imageUrl} color={0xffffffff} />
```

`rtt` (render-to-texture) elements get `color = 0xffffffff` automatically:
```typescript
if (props.rtt && !props.color) {
  props.color = 0xffffffff;
}
```

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
