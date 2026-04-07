---
description: The Config object is a mutable singleton with fields for debug, animations, fonts, focus, renderer options, shaders, and state ordering — set before importing Lightning modules
type: architecture
module: core
created: 2026-04-07
---

# Config singleton holds all runtime Lightning configuration

`Config` is a module-level object exported from `src/core/config.ts`. It is mutable and shared across the entire framework. Setting values before any Lightning modules execute ensures consistent behavior.

```ts
import { Config } from '@lightning/solid';

Config.animationsEnabled = false;
Config.debug = true;
Config.fontSettings = { fontFamily: 'Roboto', fontSize: 80 };
Config.animationSettings = { duration: 400, easing: 'ease-out' };
```

Key fields and their defaults:

| Field | Default | Effect |
|---|---|---|
| `debug` | `false` | Enable verbose node logging in dev mode |
| `focusDebug` | `false` | Enable focus manager debug output |
| `keyDebug` | `false` | Enable key handler debug output |
| `animationsEnabled` | `true` | Master switch for all animations |
| `animationSettings` | `{ duration: 250, easing: 'ease-in-out' }` | Global animation defaults |
| `fontSettings` | `{ fontFamily: 'Ubuntu', fontSize: 100 }` | Default text rendering settings |
| `focusStateKey` | `'$focus'` | State key applied when an element receives focus |
| `lockStyles` | `true` | Whether style objects are frozen |
| `stateOrder` | `[]` | Priority ordering for state style resolution |
| `throttleInput` | `undefined` | Optional input throttle in ms |
| `taskDelay` | `undefined` | Optional task queue delay |
| `setActiveElement` | noop | Callback fired on focus change |
| `convertToShader` | default implementation | Pluggable shader conversion pipeline |

Since [[dom renderer requires both build-time flag and runtime Config flag to activate]], `domRendererEnabled` on Config alone is not enough to switch renderers.

Since [[Config.focusStateKey controls which state string marks focused elements]], changing this value affects how focus-based style variants are resolved.

Since [[Config.stateOrder controls style resolution priority when multiple states are active]], setting this field is required for deterministic multi-state styling.

`Config.throttleInput` and `Config.setActiveElement` are both consumed by the focus system. Since [[throttleInput can be set globally on Config or per-element to suppress rapid key repeats]], `Config.throttleInput` acts as a global gate before any key event reaches the focus path. Since [[useFocusManager registers global keyboard listeners and returns cleanup and focusPath]], `Config.setActiveElement` must be injected before `useFocusManager` is called so focus changes trigger Solid reactivity.

---

Related Entries:
- [[activeElement is a module-level SolidJS signal tracking the currently focused ElementNode]] — the setActiveElement callback on Config bridges the focus manager to this Solid signal

Source: [[core-config]]
Domains:
- [[core guide]]
