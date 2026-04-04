# Property Values Use camelCase; Direction Strings Keep Hyphens

> Alignment and justify values are camelCase (`flexStart` not `flex-start`). Direction values like `row-reverse` retain hyphens.

**Source**: `flex-layout-engine.md` | **Severity**: important

## Detail

The lightning-tv flex engine uses camelCase for most property values, unlike the hyphenated strings used in CSS. However, `flexDirection` values that contain directional modifiers keep their hyphens.

**camelCase values** (all alignment/justify properties):
- `justifyContent`: `'flexStart'`, `'flexEnd'`, `'center'`, `'spaceBetween'`, `'spaceAround'`, `'spaceEvenly'`
- `alignItems` / `alignSelf`: `'flexStart'`, `'flexEnd'`, `'center'`

**Hyphenated values** (direction):
- `flexDirection`: `'row'`, `'column'`, `'row-reverse'`, `'column-reverse'`
- `flexWrap`: `'nowrap'`, `'wrap'`, `'wrap-reverse'`

**Other string values**:
- `display`: `'flex'`, `'block'`
- `flexBoundary`: `'contain'`, `'fixed'`
- `flexCrossBoundary`: `'fixed'`
- `direction`: `'ltr'`, `'rtl'`

## Gotchas

- Using CSS-style `'flex-start'` instead of `'flexStart'` will silently produce no alignment -- the engine will not match the string and will fall through to the default (no alignment applied).
- `spaceBetween`, `spaceAround`, `spaceEvenly` are single camelCase words with no hyphens, unlike CSS `space-between`, `space-around`, `space-evenly`.

## Related Notes

- [flex/no-flex-flow.md] -- `flexFlow` shorthand (which uses this combined string syntax in CSS) also does not exist
