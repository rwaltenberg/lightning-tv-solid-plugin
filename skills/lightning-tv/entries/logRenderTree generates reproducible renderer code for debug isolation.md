---
description: logRenderTree(node) walks from node to root and generates JavaScript code that recreates the exact node tree using renderer.createNode — useful for isolating rendering bugs
type: api
module: core
created: 2026-04-07
---

# logRenderTree generates reproducible renderer code for debug isolation

`logRenderTree(node: ElementNode)` is a debug utility that produces a self-contained JavaScript snippet capable of recreating the exact renderer node tree from the given node up to the root.

```ts
import { logRenderTree } from '@lightning/solid/core/utils';

console.log(logRenderTree(problematicNode));
```

**Output format:**
```js
function convertEffectsToShader(styleEffects) { ... }

const props0 = { /* root node props */ };
props0.parent = rootNode;
const node0 = renderer.createNode(props0);

const props1 = { /* child props */ };
props1.parent = node0;
props1.shader = convertEffectsToShader({ ... }); // if effects present
const node1 = renderer.createNode(props1);
// ... down to the target node
```

**What it includes:**
- All `_rendererProps` for each node (parent reference set to `undefined` then reassigned via variable)
- Shader configuration via a `convertEffectsToShader` helper that reconstructs `DynamicShader` effects
- Nodes in order from root to target

**What it excludes:**
- `parent` prop is stripped from JSON (set to undefined before stringify) and reconnected via code
- `shader` prop is stripped and rebuilt from `_effects` if present

This is particularly valuable when reproducing WebGL rendering issues outside of the SolidJS reactive context, as the output runs directly against the renderer API.

Since [[log utility enables per-node debug logging without global debug flag]], `logRenderTree` output is typically printed via `console.log` directly rather than through `log()` since it's an explicit debug action.

---

Source: [[core-utils]]
Domains:
- [[core guide]]
