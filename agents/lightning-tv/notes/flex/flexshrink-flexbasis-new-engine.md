# flexShrink and flexBasis Require the New Engine

> `flexShrink` and `flexBasis` are only effective when `VITE_USE_NEW_FLEX` is set. The old engine silently ignores them. `flexShrink` defaults to 0.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

The old flex engine (`flex.ts`) does not read `flexShrink` or `flexBasis` at all. Setting these properties when the old engine is active has no effect.

The new engine (`flexLayout.ts`, active when `VITE_USE_NEW_FLEX` environment variable is set) adds:

**`flexShrink`** -- default behavior is 0 (no shrinking). When available space is negative and `totalFlexShrink > 0`, the new engine applies a scaled shrink factor algorithm:
```
scaledShrinkFactor(child) = flexShrink * childMainSize
shrinkRatio = scaledShrinkFactor / totalScaledShrinkFactor
sizeReduction = shrinkRatio * |availableSpace|
newSize = max(childMainSize - sizeReduction, minDimension)
```

This differs from CSS Flexbox, where `flex-shrink` defaults to 1.

**`flexBasis`** -- accepts `number` or `'auto'`. When `'auto'` or `undefined`, it uses the child's `width`/`height`. When a number, it overrides the starting size (clamped to `minDimension`). From `flexLayout.ts`:
```ts
const isBasisAuto = flexBasis === undefined || flexBasis === 'auto';
const computedBasis = isBasisAuto ? c[dimension] || 0 : flexBasis as number;
const baseMainSize = isBasisAuto ? computedBasis : Math.max(computedBasis, c[minDimension] || 0);
```

`flexBasis` does not support CSS length values like percentages or `fit-content` -- only numbers and `'auto'`.

The `flexShrink`/`flexGrow` grow/shrink phase still requires `numProcessedChildren > 1`.

## Gotchas

- The default `flexShrink` is 0, not 1 as in CSS. Overflowing children will not shrink unless you explicitly set `flexShrink > 0`.
- `flexShrink` on the old engine is a silent no-op. Switching from new to old engine will break shrink behavior with no error.
- `flexBasis` percentage strings are not supported -- only `number` or `'auto'`.

## Related Notes

- [flex/flexgrow-two-children.md] -- `numProcessedChildren > 1` guard applies to shrink too
- [flex/container-auto-sizing.md] -- `flexBoundary` is auto-set to `'fixed'` when `flexShrink` is present
- [flex/padding-array-engine-version.md] -- other properties only available in the new engine
