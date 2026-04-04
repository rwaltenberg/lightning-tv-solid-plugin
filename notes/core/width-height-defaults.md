# Width/Height Defaults

> When w or h is NaN (never set), render() defaults to parentWidth - x or parentHeight - y. flexGrow items default to 0 instead.

**Source**: `core-rendering-nodes.md` | **Severity**: important

## Detail

When `w` or `h` is `NaN` (which happens when the property is never set), the render path interprets this as "auto-size to parent":

```ts
if (isNaN(props.w as number)) {
  props.w = node.flexGrow ? 0 : parentWidth - props.x;
}
```

This means:
- **Normal nodes**: width defaults to `parentWidth - x` (fills remaining horizontal space from position x).
- **Height**: defaults to `parentHeight - y` (fills remaining vertical space from position y).
- **flexGrow items**: width defaults to `0` (the flex algorithm will expand it).

Internally, the node stores `_calcWidth = true` / `_calcHeight = true` when the width/height was computed this way.

### Design Intent

This is by design -- it is the "fill parent" behavior. Omitting `w` and `h` is the intended way to achieve this. It is NOT a bug.

### When NaN Is Returned

Accessing `width` or `height` before rendering will return `NaN` or `undefined` because the default computation has not yet run.

## Gotchas

- Explicitly setting `w={NaN}` is fragile and undefined behavior -- either omit the property or set it to a concrete number.
- `_calcWidth` / `_calcHeight` flags are set internally -- do not rely on reading `w` before `render()` to check if width was set.
- A node with `x={400}` and no `w` inside a 1920-wide parent will have `w = 1920 - 400 = 1520`.
- flexGrow items get `w = 0` as the default so the flex algorithm can distribute space correctly.

## Related Notes

- [core/node-lifecycle.md] -- render() sequence where the default is applied (step: "Default w/h to parent dimensions minus offset")
- [constraints/unsized-text-breaks-flex.md] -- NaN dimensions on text nodes abort flex calculation
