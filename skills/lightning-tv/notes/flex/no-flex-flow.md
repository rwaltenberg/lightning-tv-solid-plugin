# flexFlow Shorthand Does Not Exist

> There is no `flexFlow` shorthand. Set `flexDirection` and `flexWrap` as separate properties.

**Source**: `flex-layout-engine.md` | **Severity**: informational

## Detail

The CSS `flex-flow` shorthand (which combines `flex-direction` and `flex-wrap` in a single property) is not supported. Both properties must be set individually on the container `ElementNode`.

`flexDirection` accepts: `'row'`, `'column'`, `'row-reverse'`, `'column-reverse'`. Default is `'row'`.

`flexWrap` accepts: `'nowrap'`, `'wrap'`, `'wrap-reverse'`. Default is `undefined` (no wrap). Note that `'wrap-reverse'` is broken in the new engine -- see `wrap-reverse-broken.md`.

## Code Example

```tsx
// WRONG: flexFlow shorthand does not exist
<view display='flex' flexFlow='row wrap' />

// CORRECT: set separately
<view display='flex' flexDirection='row' flexWrap='wrap' />
```

## Related Notes

- [flex/wrap-reverse-broken.md] -- `wrap-reverse` bug in the new engine
- [flex/camelcase-values.md] -- value naming conventions (camelCase vs CSS hyphen-case)
