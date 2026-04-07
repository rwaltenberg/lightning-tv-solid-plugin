---
description: When `delay` is set, navigation-triggered loading waits `delay` ms before expanding the offset; rapid nav cancels the pending timeout and loads synchronously instead
type: pattern
module: primitives
created: 2026-04-07
---

LazyRow and LazyColumn can debounce their navigation-triggered item loading via the `delay` prop:

```ts
delay?: number  // ms to wait before loading the next item on navigation
```

## Behavior Without delay

Immediate: each `onRight`/`onDown` event that crosses the buffer threshold instantly increments `offset` by 1.

## Behavior With delay

```ts
if (timeoutId) {
  clearTimeout(timeoutId);
  // Moving faster than delay — go synchronous immediately
  setOffset(prev => Math.min(prev + 1, maxOffset));
}

timeoutId = setTimeout(() => {
  setOffset(prev => Math.min(prev + 1, maxOffset));
  timeoutId = null;
}, props.delay);
```

- On first nav crossing the threshold: schedules a load after `delay` ms
- On next rapid nav: cancels the pending timeout and loads immediately (synchronous fallback), then schedules the next one

The synchronous fallback prevents unbounded delay accumulation when the user holds the nav key. The net effect: during slow navigation, items load with a brief pause; during fast navigation, they load immediately.

## Use Case

`delay` is useful when items are heavy to mount (e.g., images with network loads) and you want to avoid mounting items the user blasts past. Setting `delay: 200` means items only mount if the user pauses on them for 200ms.

Since [[task queue suspends on focus change and resumes when renderer is idle]], both mechanisms target the same goal: deferring work when the user is actively navigating. The task queue is focus-event-driven; `delay` is time-driven. They are complementary — `delay` prevents mounting items that scroll past quickly, while the task queue prevents eagerLoad from competing with navigation rendering.

---

Source: [[primitives-Lazy]]
Domains:
- [[performance guide]]
- [[components guide]]
