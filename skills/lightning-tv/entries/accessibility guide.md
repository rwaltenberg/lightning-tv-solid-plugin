---
description: Accessibility — speech synthesis announcer, ARIA label integration, focus announcements, multi-language support
type: moc
---

# accessibility guide

Accessibility in Lightning TV/Solid centers on the Announcer system — a singleton that integrates with focus path changes to announce content via speech synthesis or ARIA live regions. Because the renderer is WebGL-based, traditional DOM accessibility tools don't apply to rendered elements; the Announcer provides the accessibility layer.

## Core Concepts

- [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]] — the core mechanism: focuses newly entered elements, reads `announce`/`title`/`announceContext` in root-to-leaf order, with debounce and 5-minute reset timer
- [[SpeechEngine processes recursive SpeechType sequences with pause tokens retry logic and ARIA fallback]] — the low-level engine: accepts strings, arrays, Promises, functions, and `SpeechSynthesisUtterance` objects; handles `PAUSE-N` tokens and network error retry
- [[focusPath signal is a module-level reactive export updated on every active element change]] — the reactive integration point; the Announcer reads from `focusPath()` to know what was just focused

## Announcer API

- [[Announcer speak method supports append and notification modes for programmatic speech control]] — `Announcer.speak(text, { append?, notification? })` for application-triggered announcements; notification mode silences all other speech until complete
- [[Announcer loading property defers speech until all elements in the focus path are loaded]] — set `element.loading = true` while fetching, then call `Announcer.refresh()` after data arrives to trigger announcement

## ElementNode Accessibility Properties

Elements can carry speech content directly as props:

```typescript
<View
  announce="Episode 5, Breaking Bad, Season 2"    // primary speech
  announceContext="Press enter to play"            // context (read after announce)
  title="Breaking Bad"                             // fallback if announce not set
  loading={isLoading()}                            // defer announcement while loading
/>
```

`SpeechType` is flexible:
```typescript
type SpeechType = string | (() => SpeechType) | SpeechType[] | SpeechSynthesisUtterance | Promise<CoreSpeechType>
```

## Initialization

The Announcer must be configured before use:

```typescript
// In App initialization:
Announcer.setupTimers({
  focusDebounce: 400,         // ms after focus change before speaking
  focusChangeTimeout: 300000, // ms inactivity before full re-announce (default 5 min)
});
Announcer.enabled = true;
Announcer.lang = 'en-US';    // BCP 47 language tag
Announcer.aria = false;       // true to use ARIA live regions instead of Web Speech API

// Wire to focus changes:
createEffect(() => {
  Announcer.onFocusChange?.(focusPath());
});
```

## ARIA vs Web Speech

| Mode | Mechanism | Use When |
|------|-----------|----------|
| `Announcer.aria = false` (default) | `window.speechSynthesis` | Browser or TV with TTS support |
| `Announcer.aria = true` | `div#aria-parent[aria-live=assertive]` with `<span>` children | Platform screen reader integration required |

In ARIA mode, the engine adds spans to a live region div, refocuses the canvas after 100ms, and cleans up. Each span carries `lang` and `aria-label` attributes.

## Speech Features

- **`PAUSE-N` tokens**: Including `"PAUSE-2"` in a speech array pauses for 2 seconds between phrases
- **Multi-language**: Individual `SpeechSynthesisUtterance` objects carry their own `lang` and `voice`; string phrases use `Announcer.lang`
- **Voice selection**: `Announcer.voice = "voice name"` selects a specific voice from `speechSynthesis.getVoices()`
- **Network retry**: Speech network errors retry up to 3 times with exponential backoff
- **String joining**: Consecutive strings are joined with `',\b '` (word boundary) to prevent date misinterpretation (e.g., "Sun 1993" → "Sunday 1993")

## Patterns

**Announcing content that loads asynchronously:**
```typescript
<View
  announce={data()?.title || undefined}
  loading={!data()}
/>
// When data loads, call:
Announcer.refresh();
```

**Programmatic notification (high-priority, interrupts focus announcements):**
```typescript
Announcer.speak('Error: network connection lost', { notification: true });
```

**Multi-segment announcement with context:**
```typescript
<Row announce="Season 2" announceContext="Navigate with arrow keys">
  <EpisodeCard announce={episode.title} announceContext="Press enter to play" />
</Row>
```

**Customizing announcement order with functions:**
```typescript
<View announce={() => [`${title()},`, `${subtitle()}`]} />
```

## Gotchas

- **`setupTimers()` must be called first**: `Announcer.onFocusChange` is `undefined` until `setupTimers()` runs. All speech is silently skipped without it.
- **Focus diff only announces new elements**: Elements already in `prevFocusPath` are not re-announced. Call `Announcer.refresh()` to force re-announcement of the current state.
- **The 5-minute reset**: After 5 minutes of no focus changes, `prevFocusPath` is cleared. The next focus change will announce the entire focus path, not just newly entered elements.
- **ARIA mode platform notes**: The code comments uncertainty about `lang` attribute support on `<span>` elements for LG and Samsung TVs. Test on target platform.
- **Canvas refocus assumption**: ARIA mode refocuses `document.getElementById('app')?.firstChild` — assumes the Lightning canvas is the first child of `#app`.

## Open Questions

- How should `Announcer.refresh()` be called after route transitions to re-announce the new page context?
- Is there a way to suppress announcements for specific focus path elements (e.g., invisible container nodes)?
