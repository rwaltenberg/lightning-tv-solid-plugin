# Marquee

> Scrolling text component that animates overflowing text in a loop when active.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

Two components are exported: `MarqueeText` (low-level, requires explicit `clipWidth`) and `Marquee` (high-level wrapper that auto-measures its own width via `onLayout`).

**MarqueeText behavior**: Measures text width via the `loaded` event. If text overflows `clipWidth - 10px` AND `marquee` is true, creates two `<text>` nodes and animates them in a looping scroll. `scrollGap` defaults to `0.5 * clipWidth` (the gap between the end of the first text and the start of the second in the loop).

**Marquee behavior**: Auto-measures `clipWidth` from its own width via `onLayout`. Sets `clipping` on itself when marquee is active. The high-level wrapper handles the clipping container so you don't have to.

**Exports**:
```ts
export function MarqueeText(props: MarqueeTextProps): JSX.Element
export function Marquee(props: MarqueeProps): JSX.Element
```

## Props / API

```ts
interface MarqueeAnimationProps {
  delay?: number;        // default 1000ms
  speed?: number;        // default 200 px/sec
  scrollGap?: number;    // default 0.5 * clipWidth
  easing?: string;       // default 'linear'
}

interface MarqueeTextProps extends TextProps, MarqueeControlProps, MarqueeAnimationProps {
  clipWidth: number;
}

interface MarqueeProps extends NewOmit<NodeProps, 'children'>, MarqueeControlProps, MarqueeAnimationProps {
  textProps?: TextProps;
  children: TextProps['children'];
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `delay` | `number` | `1000` | Milliseconds before animation starts |
| `speed` | `number` | `200` | Scroll speed in px/sec |
| `scrollGap` | `number` | `0.5 * clipWidth` | Gap between end of first text and start of second in loop |
| `easing` | `string` | `'linear'` | CSS easing function |
| `clipWidth` | `number` | required (MarqueeText only) | Width at which to clip overflow |
| `textProps` | `TextProps` | — | (Marquee only) Props passed to inner text node |
| `children` | `TextProps['children']` | — | (Marquee only) Text content |

## Code Example

```tsx
import { Marquee } from '@lightningtv/solid/primitives';

<Marquee width={400} marquee={inFocus()} speed={200} delay={1000} textProps={{ fontSize: 28 }}>
  This is a very long title that will scroll when focused
</Marquee>
```

## Gotchas

- When using `MarqueeText` directly, you MUST provide your own clipping container. The high-level `Marquee` handles clipping automatically.
- `MarqueeText` measures text via the `loaded` event -- text width is not available synchronously.
- Overflow threshold is `clipWidth - 10px` (not exactly `clipWidth`).
- Two `<text>` nodes are created to achieve the seamless loop effect.

## Related Notes

- [api/visible.md] -- can be used to conditionally show marquee content without recreation cost
