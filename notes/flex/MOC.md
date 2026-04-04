# Flex Layout -- Engine Traps & CSS Divergences

The flex engine is a custom single-pass layout calculator. It is NOT CSS Flexbox.

## Notes

- [flexgrow-two-children](flexgrow-two-children.md) -- flexGrow is completely ignored with only 1 processable child
- [container-auto-sizing](container-auto-sizing.md) -- flexBoundary defaults to 'contain' (grows to fit children)
- [flexstart-only-auto-sizing](flexstart-only-auto-sizing.md) -- auto-sizing only runs for justifyContent='flexStart'
- [explicit-dimensions-required](explicit-dimensions-required.md) -- no intrinsic sizing; children must have width/height
- [wrap-reverse-broken](wrap-reverse-broken.md) -- wrap-reverse fails wrap check in new engine (known bug)
- [wrap-cross-size-first-child](wrap-cross-size-first-child.md) -- all wrap lines use first child's cross-axis size
- [no-align-content](no-align-content.md) -- alignContent property does not exist
- [no-flex-flow](no-flex-flow.md) -- flexFlow shorthand does not exist
- [flexshrink-flexbasis-new-engine](flexshrink-flexbasis-new-engine.md) -- only available with VITE_USE_NEW_FLEX
- [padding-array-engine-version](padding-array-engine-version.md) -- old engine: number only; new engine: [T,R,B,L] arrays
- [gap-naming-wrap-mode](gap-naming-wrap-mode.md) -- columnGap controls vertical gap in row wrap (counterintuitive)
- [textnode-skipped-in-flex](textnode-skipped-in-flex.md) -- TextNode instances excluded from flex calculation
- [nested-flexgrow-convergence](nested-flexgrow-convergence.md) -- _containsFlexGrow allows only one re-layout pass
- [camelcase-values](camelcase-values.md) -- use flexStart not flex-start; direction strings keep hyphens
- [rtl-double-reversal](rtl-double-reversal.md) -- RTL + row-reverse reverses twice = original order
