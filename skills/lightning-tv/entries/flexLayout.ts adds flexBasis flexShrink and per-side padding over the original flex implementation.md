---
description: flexLayout.ts is the enhanced implementation; it adds flexBasis, flexShrink, per-side padding arrays, and margin arrays — the older flex.ts lacks all four
type: comparison
module: core
created: 2026-04-07
---

# flexLayout.ts adds flexBasis flexShrink and per-side padding over the original flex implementation

Two flex layout implementations exist in the codebase: `flex.ts` (original) and `flexLayout.ts` (enhanced). Both export the same function signature `(node: ElementNode) => boolean`. The active one is determined by which file is imported.

## What flexLayout.ts adds

### flexBasis
Each child can declare `flexBasis: number | 'auto'`. When numeric, it overrides the child's natural dimension as the base size for flex calculations. `'auto'` (the default) uses the child's actual width/height.

### flexShrink
When children overflow the container, `flexShrink` reduces their sizes proportionally. The older `flex.ts` has no overflow handling — children simply overflow.

### Per-side padding
`flexLayout.ts` reads `paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft` individually, and also accepts `padding` as a number array `[top, right, bottom, left]`. The old `flex.ts` only reads `padding` as a single number applied uniformly.

### Margin as array
`margin` can be `[top, right, bottom, left]`. The `getArrayValue` helper resolves 2-element (alternating), 3-element (CSS shorthand), or 4-element arrays.

## What is identical

Both implementations share: flexDirection, justifyContent (all 6 values), alignItems/alignSelf (all 3 values), flexGrow, flexOrder, flexWrap/wrap-reverse, flexBoundary, container auto-resize, preFlex state preservation, _calcHeight, and Float32Array optimization.

## New engine limitations

- **`wrap-reverse` broken** — the wrapping condition (`flexWrap === 'wrap'`) excludes `wrap-reverse`, which the old engine handles correctly.
- **Cross-alignment shift** — `doCrossAlign` now receives `paddingCrossStart`, changing baseline alignment compared to the old engine.
- **No overflow warning** — the old engine's `console.warn` when flex-grow has no available space is removed, making failures silent.

## Which to use

`flexLayout.ts` should be the primary implementation for projects needing `flexShrink`, `flexBasis`, or per-side padding. However, projects using `wrap-reverse` should stick with the old engine or avoid that value. Check the active import in `elementNode.ts` or the `VITE_USE_NEW_FLEX` env var.

---

Related Entries:
- [[flexShrink distributes overflow reduction proportionally among children]] — only available in flexLayout.ts
- [[flex container resizes to content by default unless flexBoundary is fixed]] — shared behavior
- [[flex layout engine positions children using justifyContent along the main axis]] — shared behavior
- [[VITE_USE_NEW_FLEX selects between old and new flex layout engines]] — the build-time mechanism that selects which implementation runs; new-engine-only props silently no-op with old engine

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]]
