---
description: The OnEvent type maps six node lifecycle events — loaded, failed, freed, inBounds, outOfBounds, inViewport, outOfViewport — to typed handlers receiving the ElementNode and event payload
type: api
module: core
created: 2026-04-07
---

# node lifecycle events fire on loaded failed freed and bounds changes

Element nodes emit lifecycle events that developers can subscribe to. The `OnEvent` type provides typed handlers for each event.

## Event map

| Event | Payload | When |
|-------|---------|------|
| `loaded` | `NodeLoadedPayload` | Node's texture/resource has loaded |
| `failed` | `NodeFailedPayload` | Node's texture/resource failed to load |
| `freed` | `Event` | Node's renderer resources were freed |
| `inBounds` | `Event` | Node has entered the visible area |
| `outOfBounds` | `Event` | Node has left the visible area |
| `inViewport` | `Event` | Node has entered the viewport |
| `outOfViewport` | `Event` | Node has left the viewport |

## Handler signature

```typescript
type EventHandler<E extends NodeEvents> = (
  target: ElementNode,
  event?: EventPayloadMap[E],
) => void;
```

The first argument is always the `ElementNode` that fired the event. The second is the optional payload (generic `Event` for bounds events, typed payload for loaded/failed).

## OnEvent usage

```typescript
<View
  onLoaded={(node, payload) => {
    console.log('loaded', payload?.dimensions);
  }}
  onOutOfViewport={(node) => {
    node.alpha = 0; // performance: hide off-screen nodes
  }}
/>
```

## Bounds vs viewport distinction

- **inBounds/outOfBounds**: The node relative to the renderer's bounds (often the full screen)
- **inViewport/outOfViewport**: The node relative to the current viewport (scrolling area)

---

Related Entries:
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — OnEvent is part of the NodeProps composition

Source: [[core-intrinsicTypes]]
Domains:
- [[core guide]]
