---
description: The speech engine (speech.ts) accepts strings, arrays, Promises, functions, and SpeechSynthesisUtterance objects; handles PAUSE-N delay tokens, retries network errors up to 3 times, and optionally uses ARIA live regions instead of Web Speech API
type: architecture
module: primitives
created: 2026-04-07
---

# SpeechEngine processes recursive SpeechType sequences with pause tokens retry logic and ARIA fallback

`src/primitives/announcer/speech.ts` is the low-level speech engine. It takes a `SpeechType` input and processes it as a sequential series, returning a `SeriesResult` for control and observation.

## SpeechType

```typescript
type CoreSpeechType =
  | string                          // spoken literally
  | (() => SpeechType)              // lazy generator (called at processing time)
  | SpeechType[]                    // ordered sequence
  | SpeechSynthesisUtterance;       // pre-configured utterance object

export type SpeechType = CoreSpeechType | Promise<CoreSpeechType>;
```

This is fully recursive: arrays can contain Promises, functions can return arrays, etc.

## SeriesResult

```typescript
export interface SeriesResult {
  series: Promise<void>;           // resolves when all speech finishes
  readonly active: boolean;        // true while speech is in progress
  append: (toSpeak: SpeechType) => void; // add to queue without canceling
  cancel: () => void;              // stop immediately
}
```

## Processing Pipeline

Items are processed sequentially in an async loop. Before processing, consecutive leading strings are merged via `flattenStrings`:

```typescript
// "Rising Sun" + "1993" → "Rising Sun,\b 1993" (word boundary prevents date misinterpretation)
```

Processing rules by type:

| Item Type | Action |
|-----------|--------|
| `"PAUSE-N"` string | Wait `N * 1000` ms |
| regular string | `window.speechSynthesis.speak(utterance)`, retry up to 3x on network errors |
| `SpeechSynthesisUtterance` | Extract text and speak, same retry logic |
| `() => SpeechType` | Call function, recurse via `speakSeries` |
| `SpeechType[]` | Recurse via `speakSeries` |
| `Promise<...>` | `await`, then process resolved value |

## Network Error Retry

For both strings and utterances, network errors retry up to 3 times with exponential backoff (500ms, 1000ms, 1500ms):

```
error: 'network'     → retry (up to 3x with 500ms * retry_count delay)
error: 'canceled'    → silently skip (ignore)
error: 'interrupted' → silently skip (ignore)
other errors         → throw
```

## ARIA Mode

When `aria: true`:
- String phrases are NOT spoken via Web Speech API
- Instead, each phrase is accumulated as `{ text, lang }` pairs
- On series completion, a `div#aria-parent` with `aria-live="assertive"` is created (or retrieved) in the DOM
- Each phrase becomes a `<span lang="..." aria-label="...">` child
- After 100ms: ARIA div is cleaned, canvas element (`#app > firstChild`) is refocused

The `aria-live="assertive"` attribute makes screen readers interrupt current speech immediately for these announcements.

## Module-Level Singleton

The default export tracks `currentSeries` at module level. Each new call cancels the previous series:

```typescript
export default function(toSpeak, aria, lang = 'en-US', voice?) {
  currentSeries && currentSeries.cancel();
  currentSeries = speakSeries(toSpeak, aria, lang, voice);
  return currentSeries;
}
```

## `append` Behavior

`append(toSpeak)` pushes items into `remainingPhrases` while the series runs. Items are processed in order after the current phrase finishes. Since this modifies the array used by the async loop, it is safe to call at any time while `active` is true.

Since [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]] calls this engine via `Announcer.speak()`, the speech engine is the execution layer for all accessibility announcements. [[Announcer speak method supports append and notification modes for programmatic speech control]] documents the public-facing API that wraps this engine, including the `append` and `notification` modes that route through `SeriesResult.append()` and `SeriesResult.cancel()` respectively.

---

Source: [[primitives-speech]]
Domains:
- [[accessibility guide]]
