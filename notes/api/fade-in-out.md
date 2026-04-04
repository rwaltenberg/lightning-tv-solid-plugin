# FadeInOut

> Animated alpha transition wrapper that fades children in on mount and out before removal.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`FadeInOut` wraps children in `<Show when={props.when} keyed>`. On create: sets alpha to 0, then animates to 1. On destroy: sets `rtt = true` (render-to-texture), animates alpha from 1 to 0, then waits for the animation to stop before actual removal. Default duration is 250ms with 'ease-in-out' easing.

**Exports**:
```ts
export function FadeInOut(props: Props & NodeProps): JSX.Element
export function fadeIn(el: ElementNode): void
export function fadeOut(el: ElementNode): Promise<void>
export const ALPHA_NONE = { alpha: 0 }
export const ALPHA_FULL = { alpha: 1 }
```

## Props / API

```ts
interface Props {
  transition?: { duration?: number; easing?: string };
  when?: boolean;
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `when` | `boolean` | — | Controls visibility; drives the `<Show>` |
| `transition` | `{ duration?: number; easing?: string }` | `{ duration: 250, easing: 'ease-in-out' }` | Animation configuration |

Also accepts all `NodeProps`.

**Constants**:
- `ALPHA_NONE = { alpha: 0 }`
- `ALPHA_FULL = { alpha: 1 }`

**Imperative helpers**:
- `fadeIn(el)`: animates an ElementNode's alpha from 0 to 1.
- `fadeOut(el)`: returns a Promise, sets `rtt = true`, animates alpha to 0, resolves after animation stops.

## Gotchas

- On destroy, `rtt = true` is set before the fade-out animation. This enables render-to-texture so the node remains visible during the exit animation.
- Children are destroyed via `<Show>` -- after the fade-out completes. For cases where you need to keep children alive (only toggle visibility), use `Visible` instead.

## Related Notes

- [api/visible.md] -- alternative that keeps children mounted (no destruction)
- [api/image.md] -- can be combined for animated image transitions
