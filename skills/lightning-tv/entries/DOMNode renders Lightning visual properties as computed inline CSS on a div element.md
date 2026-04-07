---
description: Every prop setter calls updateNodeStyles() which rebuilds the full inline style string; complex visuals use a divBg background layer and optional divBorder layer for masks and borders
type: architecture
module: core
created: 2026-04-07
---

# DOMNode renders Lightning visual properties as computed inline CSS on a div element

The DOM renderer's `DOMNode` class maps the Lightning renderer's visual property model to HTML/CSS. Every visual property change triggers `updateNodeStyles()` which recomputes and re-applies the entire inline style as a single `setAttribute('style', ...)` call.

**Single div is not enough.** Complex visuals require up to three nested divs:

- `div` — the main container; receives position, opacity, transform, dimensions
- `divBg` — background layer (`position: absolute; z-index: -1`); created when images with tints, gradient masks, or other CSS mask effects are needed
- `divBorder` — border layer (`position: absolute; z-index: -1`); created when a mask is present AND a border is needed (they can't share the same element)

When an image source is present, an `<img>` element is created inside `divBg`.

**Transform rendering** applies in this order: translateX, translateY, rotation (in radians), scale/scaleX/scaleY. Mount offsets (`mountX * w`, `mountY * h`) are subtracted from x/y before translation. Pivot (`pivotX`, `pivotY`) sets `transform-origin` only when it differs from the 0.5/0.5 default.

**Color rendering** logic:
- Solid color → `background-color`
- Vertical gradient (colorTop ≠ colorBottom) → `linear-gradient(to bottom, ...)`
- Horizontal gradient (colorLeft ≠ colorRight) → `linear-gradient(to right, ...)`
- Both gradients → combined as multiple `background-image` layers

**Shader simulation** (via CSS):
- `radius` → `border-radius`
- `border-w` + `border-color` → `box-shadow` (inside/outside/center align via inset)
- `radial` shader → `radial-gradient()` CSS function
- `linear` shader → `linear-gradient()` CSS function

Since [[dom renderer requires both build-time flag and runtime Config flag to activate]], this class only runs when both gates are true. When running in WebGL mode, none of this code executes.

---

Source: [[dom-renderer]]
Domains:
- [[core guide]]
