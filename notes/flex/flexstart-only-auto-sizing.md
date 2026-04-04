# Auto-Sizing Only Runs for justifyContent='flexStart'

> Container dimension auto-sizing is skipped for all `justifyContent` values other than `'flexStart'`.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

The code path that calculates `calculatedSize`, updates `node[dimension]`, stores `preFlex{dimension}`, and returns `true` (triggering parent re-layout) **only exists inside the `flexStart` branch** of the main-axis positioning phase.

All other justify modes -- `flexEnd`, `center`, `spaceBetween`, `spaceAround`, `spaceEvenly` -- distribute children within the existing container size and do not update the container's dimensions. They implicitly assume `flexBoundary='fixed'` behavior.

This means: if you rely on a flex container to size itself to its content and you use any justify mode other than `flexStart`, the container will not grow to fit its children.

## Gotchas

- Setting `flexBoundary='contain'` (the default) has no effect unless `justifyContent` is `'flexStart'`.
- `spaceBetween`, `spaceAround`, and `spaceEvenly` require a fixed, known container size to compute inter-item spacing. Using them with an unsized container produces 0-space distribution.
- If you change `justifyContent` away from `flexStart` and the container was previously auto-sized, the container retains its previously computed size -- it does not shrink back.

## Related Notes

- [flex/container-auto-sizing.md] -- overall flexBoundary behavior and defaults
