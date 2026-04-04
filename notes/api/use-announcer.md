# useAnnouncer

> Accessibility announcer that reads focus-path content via SpeechSynthesis or ARIA live region.

**Source**: `ui-primitives.md` | **Severity**: important

## Detail

`useAnnouncer` watches the `focusPath` signal. On focus change, it collects `announce`, `title`, and `announceContext` from each element in the focus path. Two modes: SpeechSynthesis API or ARIA live region. Supports `PAUSE-{seconds}` strings for timed pauses between phrases. Singleton -- DO NOT call more than once.

**Export**: `export function useAnnouncer(options?): Announcer`

**Augmented ElementNode properties** (added to any Lightning node):
- `announce` -- text to announce when element is focused
- `announceContext` -- additional context (read after `announce`)
- `title` -- title text (read as part of focus announcement)
- `loading` -- loading state indicator

## Props / API

```ts
useAnnouncer(options?)
```

Options include at minimum: `{ focusDebounce: number }` (inferred from usage example).

## Code Example

```tsx
import { useAnnouncer, Announcer } from '@lightningtv/solid/primitives';

useAnnouncer({ focusDebounce: 400 });

<view announce="Play button" announceContext="Press Enter to play">
  <text>Play</text>
</view>

// Imperative:
Announcer.speak('Loading complete', { notification: true });
```

## Gotchas

- DO NOT call `useAnnouncer` more than once. It is a singleton. Duplicate calls either no-op or create duplicate listeners.
- `PAUSE-{seconds}` strings in announced content create timed pauses between phrases (e.g., `"Title PAUSE-2 Subtitle"` pauses 2 seconds).

## Related Notes

- [api/use-mouse.md] -- another input utility
