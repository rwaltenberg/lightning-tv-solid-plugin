---
description: Four per-direction transition props on NavigableProps specify which NodeStyles transition to use when navigating in each direction; overrides the container's default itemTransition
type: api
module: primitives
created: 2026-04-07
---

# NavigableProps transitionLeft right up down override animation per navigation direction

Navigable containers (Row/Column) support direction-specific animation overrides for the scroll transition:

```ts
transitionUp?: NodeStyles['transition'];
transitionDown?: NodeStyles['transition'];
transitionLeft?: NodeStyles['transition'];
transitionRight?: NodeStyles['transition'];
```

These props allow different animation durations or easing curves depending on which direction the user is navigating. For example, a Row might use a fast transition when navigating right (forward) and a slightly slower one when navigating left (backward).

The transition value is the same type as `NodeStyles['transition']` — a transition configuration accepted by the Lightning animation system.

These override `itemTransition` from `NavigableStyleProperties`, which is the default transition applied in all directions. Direction-specific transitions take precedence over the generic `itemTransition` when navigation occurs in that direction.

Since [[animatable number properties route through transition system before reaching the renderer]], the actual animation is handled by the same transition infrastructure used for all declarative animations.

---

Source: [[primitives-types]]
Domains:
- [[navigation guide]]
- [[components guide]]
