---
description: Renders FPS, memory usage, texture counts, quads, and draw calls in a fixed overlay at the right edge of a 1920px screen; requires setupFPS(root) call to activate
type: api
module: primitives
created: 2026-04-07
---

# FPSCounter component displays real-time Lightning renderer performance metrics as a debug overlay

`FPSCounter` is a development tool that renders an overlay with renderer metrics. It must be paired with `setupFPS(root)` which hooks into Lightning renderer events.

```tsx
// In app initialization
import { setupFPS, FPSCounter } from '@lightningtv/solid/primitives';

setupFPS(root);  // root is the Lightning root object

// In JSX
<FPSCounter />
```

**`setupFPS(root)`** attaches two event listeners:
- `fpsUpdate` — fires each frame; updates fps/min/max/avg signals; every 10 frames also queries `stage.txMemManager.getMemoryInfo()` for memory metrics. FPS values ≤ 5 are ignored (filters page-load spikes)
- `renderUpdate` — updates `quads` and `renderOps` signals

**Metrics displayed**:
- FPS (current, average, min, max)
- criticalThreshold, targetThreshold (MB)
- renderableMemUsed, memUsed (MB)
- Textures In Memory, Textures On Screen
- Quads, Draws (renderOps)

**Fixed positioning**: hardcoded to `x: 1900, mountX: 1` — right-aligned to a 1920px-wide display. Customize by passing position props: `<FPSCounter x={...} y={...} />`.

**Module-level signals**: all metric signals are module-scoped singletons. Multiple `<FPSCounter>` instances share the same data. `resetCounter()` clears accumulated avg/min/max stats.

---

Source: [[primitives-FPSCounter]]
Domains:
- [[components guide]]
- [[performance guide]]
