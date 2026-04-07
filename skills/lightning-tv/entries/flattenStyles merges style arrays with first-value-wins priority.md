---
description: flattenStyles recursively merges Styles objects or arrays of Styles into one object — earlier entries in the array take priority; 0 values are preserved correctly
type: api
module: core
created: 2026-04-07
---

# flattenStyles merges style arrays with first-value-wins priority

`flattenStyles(obj, result?)` is used internally to collapse composed style arrays into a single flat `Styles` object. Understanding the priority order is essential for predictable style composition.

```ts
import { flattenStyles } from '@lightning/solid';

const base = { color: 0xffffffff, width: 100 };
const override = { color: 0xff0000ff, height: 50 };

// First argument wins — base.color takes priority
flattenStyles([base, override]);
// Result: { color: 0xffffffff, width: 100, height: 50 }
```

**Priority rule:** The function guards with `result[key] === undefined` — once a key is set, it will not be overwritten. This means the first style in the array wins for any conflicting property.

**Correct 0-value handling:** The guard uses strict `=== undefined`, not falsy check. So `{ x: 0 }` will correctly prevent a later `{ x: 100 }` from overriding it. This avoids a common JS pitfall where `0` is treated as "not set."

**Recursive:**
```ts
flattenStyles([[base, override], extra]);
// Flattens nested arrays too
```

The `result` parameter enables in-place accumulation to avoid allocation:
```ts
const out: Styles = {};
flattenStyles(styles, out); // fills out directly
```

Since [[Config singleton holds all runtime Lightning configuration]] includes `lockStyles: true` which may freeze style objects before they reach flattenStyles — understand that locked styles cannot be mutated after creation.

---

Source: [[core-utils]]
Domains:
- [[core guide]]
- [[styling guide]]
