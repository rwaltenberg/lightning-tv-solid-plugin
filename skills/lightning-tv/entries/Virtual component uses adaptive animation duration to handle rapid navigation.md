---
description: `getAdaptiveDuration` shortens window-shift animation when navigation events arrive faster than the configured duration, preventing animation queuing buildup
type: pattern
module: primitives
created: 2026-04-07
---

When the user navigates rapidly (holding a directional key), Virtual components need to shift the rendered window repeatedly. If each shift triggered a full-duration animation, animations would queue up and the UI would lag far behind user input.

## The Adaptive Strategy

`getAdaptiveDuration` measures elapsed time since the last navigation event:

```ts
function getAdaptiveDuration(duration: number = 250) {
  const now = performance.now();
  const delta = now - lastNavTime;
  lastNavTime = now;
  if (delta < duration) return delta;
  return duration;
}
```

If navigation events arrive faster than `duration` ms, the animation duration is shortened to match the inter-event interval. This keeps animations in lockstep with actual navigation speed rather than falling behind.

## Animation Controller Management

Before starting a new animation, the component checks for an in-progress animation and stops it:

```ts
if (cachedAnimationController && cachedAnimationController.state === 'running') {
  cachedAnimationController.stop();
}
```

The new animation then starts from the current renderer position (not from where the last animation would have ended), preventing visual jumps from interrupted animations.

## When Animations Are Disabled

When `lng.Config.animationsEnabled` is false, the container position is updated synchronously:
```ts
this.lng[axis] = this.lng[axis]! + childSize * slice().shiftBy;
```

No duration logic runs. This path is hit in tests or when the user has disabled animations in Config.

Since [[animatable number properties route through transition system before reaching the renderer]], the container position update (x for VirtualRow, y for VirtualColumn) routes through the transition system — adaptive duration directly controls the animation config passed to `animateProp`.

---

Source: [[primitives-Virtual]]
Domains:
- [[performance guide]]
- [[components guide]]
