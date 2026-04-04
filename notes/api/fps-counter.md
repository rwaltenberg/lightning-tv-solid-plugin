# FPSCounter

> Debug overlay displaying FPS, memory, texture counts, and draw call statistics.

**Source**: `ui-primitives.md` | **Severity**: informational

## Detail

`FPSCounter` is a debug overlay that displays: FPS, AVG FPS, MIN/MAX FPS, memory thresholds, texture counts, quads, and draw calls. `setupFPS(root)` must be called with the app root to subscribe to renderer events. Singleton pattern -- only one FPSCounter per app.

**Export**: `export const FPSCounter: (props: NodeProps) => JSX.Element`

## Props / API

Accepts `NodeProps` for positioning.

**Required setup**: `setupFPS(root)` must be called with the app root before `FPSCounter` will display data.

**Displays**:
- FPS (current frames per second)
- AVG FPS (average)
- MIN/MAX FPS
- Memory thresholds
- Texture counts
- Quads
- Draw calls

## Gotchas

- `setupFPS(root)` must be called first to subscribe to renderer events -- without this call, the FPSCounter will render but show no data.
- Singleton pattern: only one FPSCounter per app.

## Related Notes

- (no related notes)
