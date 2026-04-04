# Nested flexGrow Has a One-Pass Re-layout Limit

> `_containsFlexGrow` allows only one additional child re-layout pass. Deep nesting with `flexGrow` may not converge.

**Source**: `flex-layout-engine.md` | **Severity**: important

## Detail

When a flex container contains children that are themselves flex containers with `flexGrow`, the engine needs multiple layout passes to converge (outer allocates space, inner re-lays out with new size). The `_containsFlexGrow` flag manages this, but only allows one additional pass.

The flag is a tri-state: `true`, `null`, or `undefined`.

**Flow**:
1. First `calculateFlex` call with grow items: `_containsFlexGrow` is set to `true`. Children are sized.
2. In `updateLayout()`: when `_containsFlexGrow === true`, child flex containers are iterated and re-calculated (one level deep). `this` is queued for layout again.
3. On the second `calculateFlex` call: `_containsFlexGrow` is already `true`, so it is set to `null`. This stops further passes.

From `elementNode.ts` lines 1154-1183: the additional pass iterates children and re-calculates any child flex containers, then queues `this` for layout again -- but only one time.

**Consequence**: if you have a layout like `outer flex > middle flex(flexGrow) > inner flex(flexGrow)`, only `middle` gets the re-layout benefit. `inner` may not receive its final allocated size in the same render cycle.

## Gotchas

- The `_containsFlexGrow` flag prevents infinite loops at the cost of not guaranteeing convergence for depth > 1.
- There is no error or warning when the re-layout limit is hit.
- `flexItem === false` can be used to exclude a container from the flex pass entirely if it should be positioned manually.

## Related Notes

- [flex/flexgrow-two-children.md] -- the `numProcessedChildren > 1` guard also applies to the grow/shrink triggering `_containsFlexGrow`
- [flex/container-auto-sizing.md] -- `flexBoundary` is set to `'fixed'` when grow is present, which is what creates the space children grow into
