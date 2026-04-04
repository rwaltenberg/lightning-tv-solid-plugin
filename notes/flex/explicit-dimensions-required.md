# Children Require Explicit Dimensions

> Flex children have no intrinsic sizing -- always set explicit `width`/`height` or `minWidth`/`minHeight`.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

Unlike CSS Flexbox (where the browser computes intrinsic sizes from content), the lightning-tv flex engine does not compute any intrinsic size for children. A child with `width: 0` and `height: 0` occupies no space and receives no flex-grow distribution because `childMainSize` is 0.

`minWidth` / `minHeight` are enforced in Phase 1 (child pre-processing): if `child[dimension] < child[minDimension]`, the child is clamped up to `minDimension` before layout proceeds. This makes `minWidth`/`minHeight` a valid alternative to setting an explicit `width`/`height`.

The one special case that interacts with this constraint: if a child is an `ElementText` with `.text` set but no `width` or `height`, the entire flex calculation aborts and returns `false`. This is a guard for text nodes that have not been measured yet (see `textnode-skipped-in-flex.md`).

## Code Example

```tsx
// WRONG: child has no size, occupies nothing, gets no grow space
<view display='flex' width={300} height={100}>
  <view flexGrow={1} />
</view>

// CORRECT: explicit width/height
<view display='flex' width={300} height={100} flexDirection='row'>
  <view width={50} height={50} flexGrow={1} />
  <view width={50} height={50} flexGrow={3} />
</view>

// CORRECT: minWidth/minHeight as alternative
<view display='flex' width={300} height={100} flexDirection='row'>
  <view minWidth={50} height={50} />
  <view minWidth={50} height={50} />
</view>
```

## Gotchas

- `flexGrow` does not substitute for an initial size. A child with `width=0` and `flexGrow=1` grows from a base of 0. If it is the only child, `flexGrow` is also ignored entirely (see `flexgrow-two-children.md`).
- `minWidth`/`minHeight` clamping happens in Phase 1, before grow/shrink. The clamped value becomes the `childMainSize` used for space calculations.

## Related Notes

- [flex/flexgrow-two-children.md] -- `flexGrow` also requires more than one child
- [flex/textnode-skipped-in-flex.md] -- unsized `ElementText` causes full layout abort
