# wrap-reverse is Broken in the New Flex Engine

> `flexWrap='wrap-reverse'` does not activate the wrap code path in `flexLayout.ts` -- this is likely a bug.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

The two engines handle `wrap-reverse` differently:

**Old engine (`flex.ts`, line 218)**:
```ts
node.flexWrap === 'wrap' || isWrapReverse
```
Both `'wrap'` and `'wrap-reverse'` enter the wrap code path. `isWrapReverse` is derived from `node.flexWrap === 'wrap-reverse'` and controls direction within the wrap path.

**New engine (`flexLayout.ts`, line 301)**:
```ts
node.flexWrap === 'wrap'
```
Only the literal value `'wrap'` enters the wrap code path. Because `isWrapReverse` is derived from `node.flexWrap === 'wrap-reverse'` (making `node.flexWrap === 'wrap'` false), `'wrap-reverse'` never enters the wrap path. It falls through to the non-wrap `flexStart` path instead. Wrapping does not occur.

The source document describes this as "likely a bug in the new engine."

## Gotchas

- This inconsistency exists only in `flexLayout.ts` (active when `VITE_USE_NEW_FLEX` is set).
- If you need wrap-reverse behavior and are on the new engine, you must use `flexWrap='wrap'` and manage reversal through `flexOrder` or `direction='rtl'` instead.
- Do not use `flexWrap='wrap-reverse'` when `VITE_USE_NEW_FLEX` is active -- the wrapping will silently not happen.

## Related Notes

- [flex/no-align-content.md] -- other wrap limitations
- [flex/wrap-cross-size-first-child.md] -- how wrap line height is computed once wrap is active
- [flex/flexshrink-flexbasis-new-engine.md] -- other new-engine-only behaviors
