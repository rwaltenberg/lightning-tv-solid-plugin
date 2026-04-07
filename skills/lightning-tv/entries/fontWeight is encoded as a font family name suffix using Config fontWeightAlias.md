---
description: Setting fontWeight appends a weight suffix to the font family name via Config.fontWeightAlias; if no alias exists, uses the raw value; requires matching font file naming convention
type: gotcha
module: core
created: 2026-04-07
---

# fontWeight is encoded as a font family name suffix using Config fontWeightAlias

Lightning TV does not have a CSS-style `font-weight` property. Instead, font weights are implemented by naming convention — bold fonts have a different family name (e.g., `RobotoBold` vs `Roboto`).

The `fontWeight` setter implements this:

```typescript
set fontWeight(v) {
  if (this._fontWeight === v) return;
  this._fontWeight = v;
  const weight =
    (Config.fontWeightAlias && Config.fontWeightAlias[v as string]) ?? v;
  (this.lng as any).fontFamily = `${this.fontFamily || Config.fontSettings?.fontFamily}${weight}`;
}
```

**Config.fontWeightAlias** maps weight values to suffixes:
```typescript
// Example config:
Config.fontWeightAlias = {
  bold: 'Bold',
  semibold: 'SemiBold',
  light: 'Light',
}
// Setting fontWeight = 'bold' on a node with fontFamily = 'Roboto'
// produces: lng.fontFamily = 'RobotoBold'
```

If no alias exists for the weight value, the raw value is used as the suffix directly.

**Gotcha:** This system requires font files to be registered with names that match the naming convention. If `Config.fontWeightAlias['bold'] = 'Bold'` but the font file is registered as `RobotoBold`, it works. If the font file is registered differently, text renders with a fallback font.

**Deduplication:** The setter short-circuits if `_fontWeight` already equals the new value, preventing redundant `fontFamily` reassignments.

The `fontFamily` setter simply sets `_fontFamily` and updates `lng.fontFamily` directly — no alias processing.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
