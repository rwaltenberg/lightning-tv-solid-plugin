---
description: Takes a sprite sheet URL and array of SpriteDef coordinates, returns a Record of named SubTexture instances sharing one ImageTexture upload
type: api
module: primitives
created: 2026-04-07
---

# createSpriteMap slices a sprite sheet into named Lightning SubTexture instances

`createSpriteMap` is a utility for working with texture atlases (sprite sheets). It uploads the sheet once and creates `SubTexture` descriptors for each named region.

```ts
import { createSpriteMap } from '@lightningtv/solid/primitives';

const sprites = createSpriteMap('/assets/icons.png', [
  { name: 'play',  x: 0,  y: 0, width: 48, height: 48 },
  { name: 'pause', x: 48, y: 0, width: 48, height: 48 },
  { name: 'stop',  x: 96, y: 0, width: 48, height: 48 },
]);

// Use in JSX:
<view texture={sprites['play']} width={48} height={48} />
```

**`SpriteDef` interface**:
```ts
interface SpriteDef {
  name: string | number;
  x: number;
  y: number;
  width: number;
  height: number;
}
```

**Returns**: `Record<string, SubTexture>` — keys are `SpriteDef.name` values.

**Implementation**: creates one `ImageTexture` for the whole sheet, then one `SubTexture` per definition. `SubTexture` references the parent via `texture: spriteMapTexture` plus coordinate offsets `{ x, y, w: width, h: height }`.

**GPU efficiency**: the sprite sheet image is uploaded to GPU memory once. Each `SubTexture` is a lightweight descriptor object — no additional GPU uploads. This is significantly more efficient than loading individual image files for many icons.

**Synchronous**: this function creates texture descriptors immediately — no async loading. Lightning's texture manager handles the actual GPU upload lazily when a node using the texture is rendered.

Since [[createTag renders JSX off-screen as a reusable GPU texture for efficient repeated stamping]], both createTag and createSpriteMap solve the same problem — sharing GPU textures across many instances — but with different inputs: createTag takes JSX, createSpriteMap takes image coordinates.

Since [[VirtualRow and VirtualColumn render a sliding window over large item arrays]], createSpriteMap is ideal for icon sets in large virtual lists — all icons in a row share one GPU upload regardless of how many items are rendered.

---

Source: [[primitives-utils-createSpriteMap]]
Domains:
- [[components guide]]
- [[performance guide]]
