# Suspense (Framework Primitive)

> Modified Suspense that keeps children mounted in a hidden view to preserve focus tree during loading.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

This is the framework's own `Suspense`, NOT the standard SolidJS `Suspense`. It keeps children mounted in a hidden `<view>` with `forwardFocus={0}` so the focus tree remains intact during loading. This is necessary because the standard SolidJS `Suspense` does not handle the Lightning focus system.

**Export**: `export function Suspense(props: { fallback?: JSX.Element; children: JSX.Element }): JSX.Element`

## Props / API

```ts
{
  fallback?: JSX.Element;
  children: JSX.Element;
}
```

| Prop | Type | Description |
|------|------|-------------|
| `fallback` | `JSX.Element` | Content to show while loading |
| `children` | `JSX.Element` | Content to display when ready; kept mounted in hidden view |

**Key difference from SolidJS Suspense**: Children are kept mounted in a hidden `<view>` with `forwardFocus={0}`. The focus tree remains intact during the loading state.

## Gotchas

- DO NOT use `Suspense` from `solid-js` directly. SolidJS Suspense doesn't handle Lightning focus -- the focus tree will break during loading.

```tsx
// WRONG: SolidJS Suspense doesn't handle Lightning focus
import { Suspense } from 'solid-js';

// CORRECT:
import { Suspense } from '@lightningtv/solid/primitives';
```

## Related Notes

- [api/visible.md] -- another pattern for keeping nodes alive while toggling visibility
- [api/keep-alive.md] -- preserves component instances across route navigation
