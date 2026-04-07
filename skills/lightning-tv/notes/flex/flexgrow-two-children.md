# flexGrow Requires Two or More Children

> `flexGrow` is silently ignored when there is only one processable child.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

The grow/shrink phase has a hard guard: it only executes when `numProcessedChildren > 1`. A single-child flex container will never distribute remaining space to that child via `flexGrow`, regardless of the value set. This is an intentional design decision, not a bug.

The same `numProcessedChildren > 1` guard applies to `flexShrink` (new engine).

If you need a single child to fill its parent, set the child's `width`/`height` to the parent's dimensions directly.

## Code Example

```tsx
// WRONG: flexGrow is ignored -- only one processable child
<view display='flex' width={300} height={100}>
  <view width={50} height={50} flexGrow={1} />
</view>
// child1.width stays 50, not 300

// CORRECT: two children -- flexGrow works
<view display='flex' width={300} height={100} flexDirection='row'>
  <view width={50} height={50} flexGrow={1} />
  <view width={50} height={50} flexGrow={3} />
</view>
// Available space = 300 - 100 = 200
// child1 gets +50 (1/4 of 200) -> width=100
// child2 gets +150 (3/4 of 200) -> width=200
```

## Gotchas

- The guard is on `numProcessedChildren`, not total children. Children with `flexItem=false` or `TextNode` instances are not counted. A container with three DOM children but two `flexItem=false` children still counts as one processable child.
- There is no warning or error when `flexGrow` is ignored due to a single child. It fails silently.
- **`flexGrow` + `width === 0` silently skips layout entirely.** `updateLayout()` returns early without running flex calculation if `display === 'flex' && flexGrow && width === 0` (`elementNode.ts:1158-1160`). This prevents infinite loops but means children won't be laid out at all -- no warning is emitted.

## Related Notes

- [flex/container-auto-sizing.md] -- `flexBoundary` is auto-set to `'fixed'` when `flexGrow` is present; relevant to understanding the grow space calculation
- [flex/textnode-skipped-in-flex.md] -- TextNode children are not counted toward `numProcessedChildren`
- [flex/nested-flexgrow-convergence.md] -- the `_containsFlexGrow` re-layout mechanism that gates nested grow passes
