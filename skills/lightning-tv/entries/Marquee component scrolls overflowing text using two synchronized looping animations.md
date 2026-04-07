---
description: Uses two text nodes with loop:true animations offset by textWidth+scrollGap to create seamless infinite scrolling; only scrolls when marquee prop is true and text overflows
type: api
module: primitives
created: 2026-04-07
---

# Marquee component scrolls overflowing text using two synchronized looping animations

`Marquee` (and its lower-level `MarqueeText`) provide horizontally scrolling text for TV interfaces — a common pattern for titles that don't fit their container.

```tsx
<Marquee
  width={400}
  marquee={inFocus()}
  speed={200}
  delay={1000}
  scrollGap={24}
  easing="ease-in-out"
  textProps={{ fontSize: 28 }}
>
  This is a long title that will scroll when focused
</Marquee>
```

**Key props**:
- `marquee: boolean` — master switch; scrolling only activates when `true` AND text overflows
- `speed?: number` — pixels per second (default 200)
- `delay?: number` — ms before each scroll cycle (default 1000)
- `scrollGap?: number` — space between end of text and start of next loop (default `0.5 * clipWidth`)
- `easing?: string` — default `'linear'`

**Overflow detection**: text width is measured via the `loaded` event. `isTextOverflowing = textWidth > clipWidth - 10`. Both conditions (`marquee` and overflow) must be true for scrolling to activate.

**Two-node scroll technique**: when scrolling, two `<text>` nodes animate in tandem:
- `text1`: starts at x=0, ends at x=`-(textWidth+scrollGap)`
- `text2`: starts at x=`textWidth+scrollGap`, ends at x=0
Both use `loop: true` with duration = `(textWidth + scrollGap) / speed * 1000` ms, creating the illusion of infinite seamless scrolling.

**`wasFocusedBefore` optimization**: the two scroll nodes are only rendered after `marquee` has been true at least once. This defers their creation for items never focused, since `createMemo<boolean>(p => p || marquee, false)` is sticky.

**Static fallback**: a third `<text contain='width'>` always renders when NOT scrolling, with text clipped to container width.

Since [[animatable number properties route through transition system before reaching the renderer]], the x-position loop animations on both text nodes route through the same transition infrastructure used for navigation scrolling. The `loop: true` flag is a Lightning renderer animation option that repeats the animation indefinitely.

Since [[dollar-prefix state keys in NodeStyles apply style variants based on active states]], the common pattern `marquee={inFocus()}` derives its value from a `$focus` state signal — Marquee itself is stateless but is driven by its parent's focus state.

---

Source: [[primitives-Marquee]]
Domains:
- [[components guide]]
- [[styling guide]]
