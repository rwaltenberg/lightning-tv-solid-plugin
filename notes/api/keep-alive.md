# KeepAlive

> Preserves component instances across mount/unmount cycles using a keyed Map, with router integration.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`KeepAlive` preserves component instances across mount/unmount cycles using a `Map<string, KeepAliveElement>`. Components are stored by `id` and restored when the same `id` is mounted again. `KeepAliveRoute` integrates with `@solidjs/router`'s `<Route>`, saving and restoring focus state on navigation.

**Exports**:
```ts
export const KeepAlive: (props: ParentProps<KeepAliveProps>) => JSX.Element
export const KeepAliveRoute: <S extends string>(props: ...) => JSX.Element
export const storeKeepAlive: (element: KeepAliveElement) => KeepAliveElement | undefined
export const removeKeepAlive: (id: string) => void
export const clearKeepAlive: () => void
export const keepAliveElements: Map<string, KeepAliveElement>
```

## Props / API

```ts
interface KeepAliveProps {
  id: string;
  shouldDispose?: (key: string) => boolean;
  onRemove?: ElementNode['onRemove'];
  onRender?: ElementNode['onRender'];
  transition?: ElementNode['transition'];
}
```

| Prop | Type | Description |
|------|------|-------------|
| `id` | `string` | Unique key to identify the preserved instance |
| `shouldDispose` | `(key: string) => boolean` | Optional callback to decide if an instance should be disposed |
| `onRemove` | `ElementNode['onRemove']` | Callback when element is removed |
| `onRender` | `ElementNode['onRender']` | Callback when element is rendered |
| `transition` | `ElementNode['transition']` | Transition to apply |

**Utility functions**:
- `storeKeepAlive(element)`: stores a KeepAliveElement in the map.
- `removeKeepAlive(id)`: removes an entry by id.
- `clearKeepAlive()`: clears all entries from the map.
- `keepAliveElements`: the raw `Map<string, KeepAliveElement>` for direct access.

## Code Example

```tsx
import { KeepAlive } from '@lightningtv/solid/primitives';

<KeepAlive id="home-page">
  <HomePage />
</KeepAlive>
```

## Gotchas

- `KeepAliveRoute` integrates with `@solidjs/router` and saves/restores focus state on navigation.

## Related Notes

- [api/visible.md] -- simpler show/hide without router integration
- [api/suspense-primitive.md] -- keeps children mounted during loading
