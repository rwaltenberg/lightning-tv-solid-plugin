# useHold

> Key hold detection utility that distinguishes short press (onEnter) from long press (onHold).

**Source**: `ui-primitives.md` | **Severity**: informational

## Detail

`useHold` returns `[startHold, releaseHold]` for use with key press/release events. If a key is held longer than the threshold (default 500ms), `onHold()` is called. On release: calls `onRelease()` if the key was held, or `onEnter()` if the key was not held (short press).

**Export**: `export function useHold(props): [startHold, releaseHold]`

## Props / API

```ts
useHold(props)
```

Props include:
- `onHold`: called when key is held beyond threshold
- `onEnter`: called on release if key was NOT held (short press)
- `onRelease`: called on release if key WAS held (long press)
- `threshold`: hold duration in ms (default `500`)

Returns `[startHold, releaseHold]`:
- `startHold`: call on `keydown` / press event
- `releaseHold`: call on `keyup` / release event

## Gotchas

- Default threshold is 500ms.
- On release: `onRelease()` is called if held (long press), `onEnter()` is called if not held (short press) -- they are mutually exclusive.

## Related Notes

- [api/use-mouse.md] -- mouse input utility
- [api/chain-functions.md] -- composing multiple key handlers
