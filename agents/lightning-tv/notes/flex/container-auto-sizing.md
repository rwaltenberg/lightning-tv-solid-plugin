# Container Auto-Sizing (flexBoundary)

> Unlike CSS Flexbox, the container grows to fit its children by default; set `flexBoundary='fixed'` to prevent this.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

`flexBoundary` defaults to `'contain'`, meaning the container will resize itself to wrap its children after layout. The original container size before resizing is stored in `preFlexwidth` / `preFlexheight` so that parent components (e.g., Row, Column) can reference the pre-flex size.

This is the **opposite of CSS Flexbox**, where the container never grows automatically.

When any child has `flexGrow > 0` (or `flexShrink > 0` in the new engine), `flexBoundary` is automatically forced to `'fixed'`. This prevents auto-sizing from eliminating the available space that grow/shrink needs to work with.

Auto-sizing only runs during `justifyContent='flexStart'` (the default). All other justify modes (`flexEnd`, `center`, `spaceBetween`, `spaceAround`, `spaceEvenly`) assume a fixed container size. See `flexstart-only-auto-sizing.md`.

The cross-dimension auto-sizing for row containers is controlled separately by `flexCrossBoundary`. Setting `flexCrossBoundary='fixed'` suppresses auto-calculation of the container's height.

## Code Example

```tsx
// Default: container grows to fit children (flexBoundary='contain' implicit)
<view display='flex'>
  <view width={100} height={100} />
  <view width={100} height={100} />
</view>
// container.width becomes 200; container.preFlexwidth stores original width

// Fixed: container does not grow
<view display='flex' width={300} flexBoundary='fixed'>
  <view width={100} height={100} />
  <view width={100} height={100} />
</view>
// container.width stays 300
```

## Gotchas

- `flexBoundary` is automatically set to `'fixed'` when `flexGrow` is present on any child. You cannot use `flexBoundary='contain'` together with `flexGrow`.
- Auto-sizing only runs for `justifyContent='flexStart'`. Do not rely on it with other justify modes.
- The container's `width`/`height` is mutated in place. `preFlexwidth` / `preFlexheight` preserve the original values.

## Related Notes

- [flex/flexstart-only-auto-sizing.md] -- auto-sizing is gated on `justifyContent='flexStart'`
- [flex/flexgrow-two-children.md] -- `flexGrow` triggers the auto `'fixed'` override
