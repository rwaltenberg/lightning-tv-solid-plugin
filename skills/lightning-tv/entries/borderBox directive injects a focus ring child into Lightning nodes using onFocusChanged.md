---
description: Solid directive use:borderBox that creates/destroys a border child element reactively on focus changes; border auto-sizes to parent with configurable space offset
type: api
module: primitives
created: 2026-04-07
---

# borderBox directive injects a focus ring child into Lightning nodes using onFocusChanged

`borderBox` is a Solid directive that adds a visible focus ring to any Lightning `view` or `text` element. The ring is created on focus and destroyed (with fade-out animation) on blur.

```tsx
import { borderBox } from '@lightningtv/solid/primitives';

// Default styling
<view use:borderBox />

// Custom border properties
<view use:borderBox={{ borderSpace: 10, border: { color: 0xff6600, width: 3 } }} />
```

**Default style** (`BorderBoxStyle`):
```ts
{
  alpha: 0.01,
  borderSpace: 6,
  borderRadius: 20,
  border: { color: 0xffffff, width: 2 },
  transition: { alpha: { duration: 100, easing: 'linear' } },
}
```

**On focus**: creates the border `<view>` using `runWithOwner(owner, ...)` (preserving reactive context), inserts it via `insertNode(el, border)`, then calls `border.render()` to immediately render it.

**On blur**: calls `border.destroy()` which triggers `onDestroy` on the border element — the border fades its alpha to 0 and waits before the node is removed.

**Border sizing**: in `onCreate`, reads `el.parent.width/height` and sets the border's dimensions to `parent + 2*borderSpace`, positioned at `x: -borderSpace, y: -borderSpace`.

**Restriction**: can only be used on native `view` and `text` elements, not on custom component roots. Since [[insertChild mirrors DOM insertBefore and removes node from prior parent enabling node moves]], `insertNode` is the correct API for dynamically adding the border child post-render.

---

Source: [[primitives-borderBox]]
Domains:
- [[components guide]]
- [[focus guide]]
