---
description: Accepts #RRGGBB, #RRGGBBAA, 0x-prefixed hex, 6-char bare hex, or existing numbers; outputs a 32-bit RGBA integer; returns 0x00000000 for unknown inputs
type: api
module: utils
created: 2026-04-07
---

# hexColor converts string color formats to RGBA numbers for the Lightning renderer

The Lightning renderer expects colors as 32-bit RGBA integers. `hexColor()` converts common string formats into this representation:

```ts
export function hexColor(color: string | number = ''): number {
  if (isInteger(color)) return color;

  if (typeof color === 'string') {
    if (color.startsWith('#')) {
      // #RRGGBB → 0xRRGGBBff (alpha appended)
      // #RRGGBBAA → converted as-is
      return Number(color.replace('#', '0x') + (color.length === 7 ? 'ff' : ''));
    }
    if (color.startsWith('0x')) return Number(color);
    // 6-char bare hex → append ff
    return Number('0x' + (color.length === 6 ? color + 'ff' : color));
  }
  return 0x00000000;
}
```

Format lookup:
- `#RRGGBB` — 7 chars → alpha ff appended → `0xRRGGBBff`
- `#RRGGBBAA` — 9 chars → no appending → `0xRRGGBBAA`
- `0xRRGGBBAA` — already hex number string → `Number()` converts
- `RRGGBB` (6-char bare) → treated as 6-char → appends ff
- Existing number → returned as-is via `isInteger()` check
- Unknown/empty → `0x00000000` (transparent black)

**Gotcha:** A 6-char string that happens to be `#RRGGBB` without the `#` has its `ff` appended correctly. But an 8-char bare hex without `0x` prefix will NOT have `ff` appended — it passes through `Number('0x' + input)` which interprets it as RGBA directly.

This function is exported from `utils.ts` and is the standard way to pass colors from JSX props (which may be strings) into style properties or shader props.

**Convention:** The codebase universally uses numeric `0xRRGGBBAA` format. String formats work but are not idiomatic. Named CSS colors (e.g., `'red'`) are not supported — they fall through to `0x00000000`.

---

Source: [[utils]]
Domains:
- [[core guide]]
- [[styling guide]]
