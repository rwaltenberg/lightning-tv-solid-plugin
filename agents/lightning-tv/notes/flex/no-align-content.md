# alignContent Does Not Exist

> There is no `alignContent` property. Wrapped lines are always stacked without space distribution between them.

**Source**: `flex-layout-engine.md` | **Severity**: important

## Detail

The lightning-tv flex engine does not implement `alignContent`. In wrapped layouts, lines are always stacked from the start (or end for `wrap-reverse`) with no ability to distribute remaining cross-axis space between lines.

The only gap control between wrapped lines is via `rowGap` / `columnGap` (with counterintuitive naming -- see `gap-naming-wrap-mode.md`).

## Gotchas

- Do not expect `stretch`, `space-between`, `space-around`, or `space-evenly` line distribution. These CSS `align-content` values have no equivalent.
- Lines are always tightly packed using the first child's cross-size as the uniform line height (see `wrap-cross-size-first-child.md`).

## Related Notes

- [flex/wrap-cross-size-first-child.md] -- why per-line sizing is already limited
- [flex/gap-naming-wrap-mode.md] -- how to add gaps between wrapped lines
- [flex/wrap-reverse-broken.md] -- other wrap limitations
