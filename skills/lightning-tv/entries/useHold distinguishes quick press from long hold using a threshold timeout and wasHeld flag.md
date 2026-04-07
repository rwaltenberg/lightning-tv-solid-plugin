---
description: useHold returns [startHold, releaseHold] handlers that use a setTimeout to determine if a keydown-keyup pair is a quick press (onEnter) or a sustained hold (onHold + onRelease)
type: api
module: primitives
created: 2026-04-07
---

# useHold distinguishes quick press from long hold using a threshold timeout and wasHeld flag

`useHold` is a composable primitive for per-direction hold detection, independent of the global key hold map.

```typescript
export function useHold(props: UseHoldProps): [startHold: () => boolean, releaseHold: () => boolean]

type UseHoldProps = {
  onHold: () => void;              // fires after holdThreshold ms of continuous press
  onEnter: () => void;             // fires on quick press (released before threshold)
  onRelease?: () => void;          // fires when a held key is finally released
  holdThreshold?: number;          // default 500ms
  performOnEnterImmediately?: boolean; // default false
};
```

## Behavioral Matrix

| `performOnEnterImmediately` | Quick press (release before threshold) | Long hold (release after threshold) |
|-----------------------------|---------------------------------------|--------------------------------------|
| `false` (default)           | `onEnter()` fires on keyup            | `onHold()` during hold, `onRelease()` on keyup |
| `true`                      | `onEnter()` fires on keydown          | `onEnter()` on keydown, `onHold()` during hold, `onRelease()` on keyup |

## Internal State Machine

Two closure-scoped variables track state:
- `holdTimeout` — The pending `setTimeout` handle (`-1` when no hold is in progress)
- `wasHeld` — `true` if the timeout fired before `releaseHold` was called

`startHold()` guards with `holdTimeout === -1` to prevent duplicate timers. It returns `true` unconditionally to stop key propagation up the focus chain.

`releaseHold()` returns `true` unconditionally (swallows release events from propagating). If released before the timeout fires, it calls `onEnter()` (unless `performOnEnterImmediately` already did so). If released after the timeout fired (`wasHeld === true`), it calls `onRelease?.()`.

## Usage Pattern

```typescript
const [holdRight, releaseRight] = useHold({
  onHold: handleHoldRight,
  onEnter: handleOnRight,
  onRelease: handleReleaseHold,
  holdThreshold: 200,
  performOnEnterImmediately: true,
});

<View onRight={holdRight} onRightRelease={releaseRight} />
```

Multiple calls to `useHold` are independent — each has its own `holdTimeout` and `wasHeld`. Different directions (Left, Right, Up, Down) can each have their own hold behavior.

Since [[key hold detection fires after 500ms and cancels if key is released early]] describes the global hold detection in the core, `useHold` provides the same pattern but at the component level without requiring global key map configuration.

---

Source: [[primitives-useHold]]
Domains:
- [[input guide]]
