---
description: DOM renderer has its own animation loop using rAF; supports delay, repeat, loop, easing (linear/ease-in/ease-out/ease-in-out), and a waitUntilStopped() promise
type: architecture
module: core
created: 2026-04-07
---

# DOM renderer AnimationController interpolates numeric props using requestAnimationFrame

The DOM renderer implements animation independently of the WebGL renderer, using `requestAnimationFrame` rather than the Lightning animation manager.

`AnimationController` stores `propsStart` and `propsEnd` as numeric snapshots, captured at construction time:

```ts
for (let [prop, value] of Object.entries(props)) {
  if (value != null && typeof value === 'number') {
    this.propsStart[prop] = (node.props as any)[prop];
    this.propsEnd[prop] = value;
  }
}
```

Only numeric props are animated. Non-numeric props (strings, objects) are silently ignored.

The animation loop runs per-frame. For each active task:
1. Checks if still in the delay period (skips if so)
2. Computes `t = activeTime / duration`
3. Applies easing to `t`
4. For each prop: interpolates between start and end using `interpolateProp()`
5. Calls `updateNodeStyles()` to apply the updated props to the DOM

Color props are detected by name prefix (`color`): they use per-channel RGBA interpolation. All other numeric props use simple linear interpolation.

**Easing functions:**
- `linear` — `t`
- `ease-in` — `t * t`
- `ease-out` — `t * (2 - t)`
- `ease-in-out` — cubic blend (two piecewise equations)
- Custom function — `easing(progress)` called directly

**Lifecycle methods:** `start()`, `pause()`, `stop()`, `restore()` (no-op), `waitUntilStopped()` returns a Promise. This matches the `IAnimationController` interface from `@lightningjs/renderer`.

Since [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]], the DOM renderer's animate method satisfies the same interface, making animation code portable between renderers.

---

Source: [[dom-renderer]]
Domains:
- [[core guide]]
