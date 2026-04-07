---
description: fontWeightAlias on Config translates named font weights (thin, light, regular, bold, etc.) to numeric values used by the renderer — empty string means use default
type: api
module: core
created: 2026-04-07
---

# Config.fontWeightAlias maps named weights to numeric values

`Config.fontWeightAlias` is a `Record<string, number | string>` that maps human-readable weight names to the values passed to the renderer. Default mapping:

```ts
fontWeightAlias: {
  thin: 100,
  light: 300,
  regular: '',   // empty string = use default (no explicit weight)
  400: '',       // numeric key also mapped
  medium: 500,
  bold: 700,
  black: 900,
}
```

Empty string values (`''`) mean "use the renderer default" rather than specifying a weight. This allows `regular` weight to fall through to the font's natural rendering.

You can extend or override this to add custom weight names used by your font family:

```ts
Config.fontWeightAlias = {
  ...Config.fontWeightAlias,
  semibold: 600,
  extralight: 200,
};
```

Since [[Config singleton holds all runtime Lightning configuration]], this mapping applies globally to all text elements.

---

Source: [[core-config]]
Domains:
- [[core guide]]
