# Visible

> Visibility toggle that hides children without destroying them, avoiding expensive WebGL texture reallocation.

**Source**: `ui-primitives.md` | **Severity**: critical

## Detail

Unlike SolidJS `<Show>`, `Visible` does NOT destroy children when `when` becomes falsy. Instead, it sets `hidden = true` on child `ElementNode`s. Children are created once (in a `createRoot`) on the first truthy `when`, then toggled via the `hidden` property. This is critical for TV UIs where Lightning node creation is expensive due to WebGL texture allocation.

**Export**: `export function Visible<T>(props): JSX.Element`

## Props / API

```ts
function Visible<T>(props: {
  when: T | undefined | null | false;
  keyed?: boolean;
  children: JSX.Element;
}): JSX.Element
```

| Prop | Type | Description |
|------|------|-------------|
| `when` | `T \| undefined \| null \| false` | Controls visibility |
| `keyed` | `boolean` | Whether to key on the `when` value |
| `children` | `JSX.Element` | Children to show/hide |

## Code Example

```tsx
import { Visible } from '@lightningtv/solid/primitives';

<Visible when={showDetails()}>
  <view width={500} height={300}>
    <text>Details panel</text>
  </view>
</Visible>
```

Use `Visible` instead of `Show` when you want to hide/show without destroying the Lightning nodes.

## Gotchas

- Children are created ONCE on the first truthy `when`, then only toggled. They will NOT be recreated if `when` oscillates.
- DO NOT use `<Show>` from SolidJS for Lightning nodes that should persist. `<Show>` destroys and recreates nodes (expensive WebGL texture allocation). `<Visible>` only toggles `hidden` (cheap).

## Related Notes

- [api/fade-in-out.md] -- uses `<Show>` internally (destroys children) -- use when destruction is desired
- [api/keep-alive.md] -- preserves component instances across route navigation
