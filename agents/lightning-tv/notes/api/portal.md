# Portal

> Renders children into a different part of the Lightning scene graph by ID lookup.

**Source**: `ui-primitives.md` | **Severity**: informational

## Detail

`Portal` renders children into a different part of the Lightning scene graph. The `mount` prop is a string ID used to find the target node via `rootNode.searchChildrenById(mount)`. The component itself returns `null`.

**Export**: `export function Portal(props: { mount?: string; children: JSX.Element }): null`

## Props / API

```ts
{
  mount?: string;       // string ID of target node
  children: JSX.Element;
}
```

| Prop | Type | Description |
|------|------|-------------|
| `mount` | `string` | ID of the target Lightning node to mount children into |
| `children` | `JSX.Element` | Content to render in the target node |

The target node is found via `rootNode.searchChildrenById(mount)` -- it searches the scene graph by the `id` property on Lightning nodes.

## Code Example

```tsx
import { Portal } from '@lightningtv/solid/primitives';

<Portal mount="overlay-container">
  <view width={1920} height={1080} color={0x00000099}>
    <text>Modal Content</text>
  </view>
</Portal>
```

## Gotchas

- `mount` is a string ID matched against Lightning node `id` properties, NOT HTML DOM IDs.
- Portal returns `null` -- it produces no output at the call site.
- The target must already exist in the scene graph when Portal renders.

## Related Notes

- [api/keep-alive.md] -- another pattern for managing scene graph placement
