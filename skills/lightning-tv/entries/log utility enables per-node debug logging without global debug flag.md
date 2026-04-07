---
description: The log() utility logs only in dev mode and only when Config.debug is true OR the target node has a debug property set — enabling targeted per-node tracing
type: api
module: core
created: 2026-04-07
---

# log utility enables per-node debug logging without global debug flag

The `log(msg, node, ...args)` utility in `utils.ts` provides conditional debug logging with per-node granularity.

```ts
import { log } from '@lightning/solid/core/utils';

// Logs only if isDev AND (Config.debug OR node.debug OR args[0].debug)
log('Rendering node', myNode, extraContext);
```

**Activation conditions (all require `isDev`):**
1. `Config.debug === true` — global debug mode, logs everything
2. `node.debug` is truthy — per-node flag, logs only for this node
3. `args[0]?.debug` is truthy — first extra argument's debug flag

This allows enabling verbose logging for a single problematic node without flooding the console:

```ts
// Enable logging only for this specific element:
myNode.debug = true;
// Now all log() calls with myNode as the second argument will output
```

**Important:** The `isDev` gate means this produces zero output in production builds, even if `Config.debug` is set (which would be unusual in production anyway).

Since [[Config singleton holds all runtime Lightning configuration]] provides `Config.debug`, the global flag is the simplest way to enable broad logging during development.

---

Source: [[core-utils]]
Domains:
- [[core guide]]
