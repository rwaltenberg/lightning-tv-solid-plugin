---
description: handleNavigation merges a direction-specific transition into el.transition on each key press, enabling asymmetric easing for forward vs backward movement
type: architecture
module: primitives
created: 2026-04-07
---

# Row and Column apply direction-specific transitions on each navigation key press

Row and Column set four directional transition props (`transitionLeft`, `transitionRight`, `transitionUp`, `transitionDown`) at mount. Each time a navigation key is pressed, `handleNavigation` merges the matching transition into the live `el.transition` object:

```ts
export function handleNavigation(direction) {
  return function () {
    const el = this as NavigableElement;
    const transition = direction === 'left' ? el.transitionLeft : ...;

    if (transition) {
      el.transition = {
        ...currentTransition,    // preserve other axes
        ...(transition as object), // overlay direction-specific config
      };
    }

    return moveSelection(el, delta);
  };
}
```

Row's default directional transitions:
```ts
transitionLeft  = { x: { duration: 180, easing: 'cubic-bezier(0.4, 0, 0.2, 1)' } }  // "back"
transitionRight = { x: { duration: 180, easing: 'cubic-bezier(0.2, 0, 0, 1)' } }     // "forward"
```

Column's default directional transitions:
```ts
transitionUp   = { y: { duration: 300, easing: 'cubic-bezier(0.3, 0, 0.2, 1)' } }
transitionDown = { y: { duration: 300, easing: 'cubic-bezier(0.2, 1, 0.8, 1)' } }
```

Both Row and Column initialize with `transition={/* @once */ {}}` — an empty transition. The directional animations only activate as keys are pressed. This means the first render has no transition applied, which prevents animation on initial layout.

**Custom transitions:** Override any directional prop to change the animation:
```tsx
<Row
  transitionLeft={{ x: { duration: 400, easing: 'ease-in' } }}
  transitionRight={{ x: { duration: 200, easing: 'ease-out' } }}
/>
```

Since [[Row and Column use @once annotations so handler props cannot be updated after mount]], the default transition values for each direction are frozen at mount. The mechanism of merging into `el.transition` on each keypress still works, but the values being merged are the ones provided at render time.

---

Source: [[primitives-handleNavigation]]
Domains:
- [[navigation guide]]
- [[styling guide]]
