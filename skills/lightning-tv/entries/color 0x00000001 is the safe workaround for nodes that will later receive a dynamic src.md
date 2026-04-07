---
description: Setting color to 0x00000001 (near-transparent) prevents the src setter from auto-applying 0xffffffff white, avoiding a white flash before a dynamic image loads
type: gotcha
module: core
status: preliminary
created: 2026-04-07
---

# color 0x00000001 is the safe workaround for nodes that will later receive a dynamic src

When `src` is set dynamically after a node has rendered, the `src` setter checks `!this.color` and auto-applies white (`0xffffffff`) if the check passes. Since the default color is `0x00000000` (transparent, which is falsy in JavaScript), this auto-apply fires for any node that hasn't had an explicit color set.

The workaround is to pre-set `color` to `0x00000001` — a near-black color with alpha=1 that is visually transparent on most content but truthy in JavaScript, blocking the unwanted white auto-apply.

## Why this works

```typescript
// elementNode.ts — src setter
set src(src) {
  if (typeof src === 'string') {
    this.lng.src = src;
    if (!this.color && this.rendered) {  // !0x00000000 === true (fires)
      this.color = 0xffffffff;           // unwanted white flash
    }
  }
}
```

`0x00000000` is the integer `0`, which is falsy. `!0` is `true`, so the auto-apply fires.

`0x00000001` is the integer `1`, which is truthy. `!1` is `false`, so the auto-apply is blocked.

The color `0x00000001` is RGBA `(0, 0, 0, ~0.004)` — effectively invisible to the user before the image loads, but it satisfies the JavaScript truthiness check.

## When you need this

```tsx
// Pattern: image src set dynamically (e.g., from a signal or after data fetch)
const [imgSrc, setImgSrc] = createSignal<string | undefined>();

// Without workaround — white flash as src setter auto-applies 0xffffffff
<view width={200} height={120} src={imgSrc()} />

// With workaround — near-transparent placeholder, no white flash
<view width={200} height={120} color={0x00000001} src={imgSrc()} />
```

Alternatively, always provide the fully opaque white explicitly:

```tsx
// Also correct — explicit white means the src setter condition is already blocked
<view width={200} height={120} color={0xffffffff} src={imgSrc()} />
```

The difference: `0xffffffff` is always white (visible before src loads); `0x00000001` is effectively transparent (invisible before src loads). Use whichever matches your intended pre-load appearance.

## Status note

The value `0x00000001` as a named workaround does not appear in the framework source code — this entry documents a community-observed pattern. The mechanism is verified from source (see [[element nodes default color to transparent unless src or explicit color is set]]). Mark this entry preliminary until confirmed in official documentation or test fixtures.

---

Related Entries:
- [[element nodes default color to transparent unless src or explicit color is set]] — foundation: explains the default transparent color and the src setter auto-apply behavior in detail
- [[Image component shows placeholder while src loads and fallback on error]] — the framework's built-in solution for image loading states; handles this concern automatically

Domains:
- [[core guide]]
