# createTag Cleanup Required

> createTag creates off-screen nodes with preventCleanup: true. Must call .destroy() to free GPU textures.

**Source**: `solidjs-integration.md` | **Severity**: important

## Detail

`createTag` creates off-screen nodes with `preventCleanup: true`. This prevents the node deletion queue from automatically destroying them when they are removed from the tree.

Because `preventCleanup: true` bypasses the normal deletion queue mechanism, these nodes hold GPU texture references indefinitely until explicitly destroyed.

**You must call `.destroy()` on nodes created with `createTag`** when they are no longer needed, to free GPU texture memory.

### preventCleanup

The `preventCleanup` property (non-animating, proxied to `this.lng[key]`) signals to the node deletion queue that the node should not be automatically destroyed. Setting `preventCleanup: true` means `_queueDelete` will not trigger `destroy()`.

## Code Example

```tsx
// createTag creates an off-screen node
const tag = createTag({ /* props */ });

// WRONG -- forgetting to destroy leaks GPU textures
// (no cleanup happens automatically)

// CORRECT -- always destroy when done
onCleanup(() => {
  tag.destroy();
});
```

## Gotchas

- `preventCleanup: true` is a permanent flag -- the node will never be auto-destroyed by the deletion queue.
- GPU texture leaks are silent and can accumulate over the lifetime of the application.
- Use `onCleanup` from SolidJS to tie tag destruction to component lifecycle.

## Related Notes

- [core/node-deletion-queue.md] -- how the deletion queue normally handles destruction
- [core/node-lifecycle.md] -- destroy() method and GPU resource cleanup
