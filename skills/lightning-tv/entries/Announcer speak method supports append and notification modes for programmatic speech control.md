---
description: Announcer.speak() cancels current speech by default; append mode adds to ongoing speech; notification mode suppresses all other voice output until complete then refreshes
type: api
module: primitives
created: 2026-04-07
---

# Announcer speak method supports append and notification modes for programmatic speech control

`Announcer.speak()` is the programmatic entry point for triggering speech outside of automatic focus announcements.

```typescript
Announcer.speak(
  text: SpeechType,
  options?: { append?: boolean; notification?: boolean }
): SeriesResult
```

## Default Behavior (no options)

Cancels any currently playing speech and starts the new text immediately. Returns a `SeriesResult` for tracking completion.

## `append: true`

If `currentlySpeaking?.active` is true, appends `text` to the ongoing speech queue without interrupting. If no speech is active, behaves like a normal speak call.

```typescript
Announcer.speak('Loading...'); // starts speech
Announcer.speak('done', { append: true }); // appends "done" after "Loading..."
```

## `notification: true`

Used for high-priority one-time announcements (alerts, errors, confirmations):

1. Speaks the text normally (canceling current speech first).
2. Sets `voiceOutDisabled = true` — all further speech (including focus announcements) is silently blocked.
3. When the notification speech finishes (`.finally()`), sets `voiceOutDisabled = false`.
4. Calls `Announcer.refresh()` to re-announce the current focus state.

This ensures a notification is never interrupted by a focus change, and the app resumes normal announcements afterward.

```typescript
Announcer.speak('Error: cannot connect', { notification: true });
// All focus announcements are silenced until this finishes
```

## `cancel()`

```typescript
Announcer.cancel();
```

Immediately cancels the current `SeriesResult` (delegates to `currentlySpeaking.cancel()`).

## `refresh(depth?)`

```typescript
Announcer.refresh(depth = 0);
```

Clears `prevFocusPath` to `depth` elements and triggers `onFocusChange` with the current focus path. Use to force re-announcement after content loads:

```typescript
// After async content loads:
myElement.loading = false;
Announcer.refresh(); // re-announces everything
```

Since [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]], `refresh` works by resetting the diff baseline — a cleared `prevFocusPath` means all current path elements are "new" and will be announced.

---

Source: [[primitives-announcer]]
Domains:
- [[accessibility guide]]
