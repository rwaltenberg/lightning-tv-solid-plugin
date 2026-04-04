# textAlign Requires contain

> render() warns 'Text align requires contain'. Must set contain="width" or contain="both" with textAlign.

**Source**: `solidjs-integration.md`, `core-rendering-nodes.md` | **Severity**: important

## Detail

The `render()` method in `ElementNode` explicitly warns: `'Text align requires contain'` when `textAlign` is set without `contain`.

The Lightning renderer's text alignment only works within a contained text box. Without `contain`, the renderer has no bounding box to align the text within, so `textAlign` has no effect.

### Valid contain Values for textAlign

- `contain="width"` -- constrains horizontally, allowing text to wrap or clip. Requires `w` to be set.
- `contain="both"` -- constrains both horizontally and vertically. Requires both `w` and `h`.

## Code Example

```tsx
// WRONG -- textAlign without contain
<text textAlign="center">Hello</text>

// CORRECT -- contain="width" with explicit width
<text textAlign="center" contain="width" w={400}>Hello</text>

// CORRECT -- contain="both" for full containment
<text textAlign="right" contain="both" w={400} h={50}>Hello</text>
```

## Gotchas

- The warning fires during `render()` -- it is a runtime warning, not a TypeScript compile-time error.
- `contain` without a corresponding `w` (for `contain="width"`) or `w` + `h` (for `contain="both"`) may produce unexpected results.
- This constraint applies to `<text>` elements only (`_type: 'textNode'`).

## Related Notes

- [core/node-lifecycle.md] -- render() sequence where the warning fires
- [api/element-node.md] -- contain and textAlign property definitions
