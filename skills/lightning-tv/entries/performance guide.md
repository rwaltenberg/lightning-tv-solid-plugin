---
description: Performance optimization — virtual scrolling, lazy loading, Preserve/Visible components, task scheduling queue
type: moc
---

# performance guide

Performance in TV apps is critical — limited hardware, 60fps target. The framework provides virtual scrolling (only render visible items), lazy loading (defer rendering), component preservation (hide instead of destroy), and a task scheduling queue.

## Core Concepts

- [[VirtualRow and VirtualColumn render a sliding window over large item arrays]] — the primary virtualization approach: only `displaySize + bufferSize` items are mounted at any time, with the window shifting as the user navigates
- [[VirtualRow and VirtualColumn track absolute cursor and relative selected as separate signals]] — two-signal design separating absolute position in the full array from position in the rendered slice; both must be synchronized on every navigation
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — the 2D equivalent of Virtual; items per row is fixed, buffer is row-count-based
- [[LazyRow and LazyColumn incrementally render items one per frame until upCount is reached]] — deferred rendering that starts the list interactive immediately and loads remaining items in the background
- [[Visible component toggles ElementNode hidden instead of destroying and recreating children]] — within-page conditional rendering that trades memory for faster show/hide cycles
- [[Preserve component marks an ElementNode as preserve=true to survive route transitions]] — renderer-level node preservation across route changes; the foundation for instant back-navigation
- [[createInfiniteItems accumulates paginated data into a single reactive array]] — data primitive for infinite-scroll; accumulates pages and detects end automatically
- [[lazy function loads a component asynchronously and exposes a preload method for eager fetching]] — code-splitting primitive; renders null until the import resolves, with `.preload()` for prefetching
- [[scheduleTask enables deferred background work with high and low priority]] — public API for app-level background tasks; pauses automatically during navigation
- [[task queue suspends on focus change and resumes when renderer is idle]] — the suspension mechanism: activeElement() change → tasksEnabled=false; renderer 'idle' → re-enable
- [[preserve property prevents node destruction by holding the delete counter at zero]] — _queueDelete=0 makes a node immune to SolidJS reconciler destruction

## Patterns

### Virtual + Infinite Scroll

Pair [[VirtualRow and VirtualColumn render a sliding window over large item arrays]] with [[createInfiniteItems accumulates paginated data into a single reactive array]] using the `onEndReached` callback. Since [[Virtual onEndReached fires when cursor moves within threshold of the last item]], set `onEndReachedThreshold` high enough to fetch the next page before the user reaches the end:

```ts
const [items, { setPage, end }] = createInfiniteItems(fetcher);
<VirtualRow each={items()} onEndReachedThreshold={5} onEndReached={() => !end() && setPage(p => p + 1)}>
```

### Lazy for Initial Load Performance

Use [[LazyRow and LazyColumn incrementally render items one per frame until upCount is reached]] when an entire row must render before focus arrives. Set `eagerLoad: true` to pre-render everything in the background using the task queue. Use `delay` (see [[Lazy delay prop debounces item loading during rapid navigation]]) when items are heavy (images) and the user may scroll past them quickly.

### Preserve for Route Performance

Wrap expensive pages with [[Preserve component marks an ElementNode as preserve=true to survive route transitions]] to keep the renderer node alive. Navigation back is instant — no remounting, no image reloads. The Solid reactive scope must stay alive for preservation to persist.

### Virtual vs Lazy: When to Use Each

| Scenario | Use |
|----------|-----|
| Very large lists (500+) | Virtual — items outside window are unmounted |
| Moderate lists with slow mount | Lazy — items mount once and stay |
| List that needs circular navigation | VirtualRow/VirtualColumn with `wrap: true` |
| 2D grid layout | VirtualGrid |
| Items grow from a paginated API | Lazy or Virtual + createInfiniteItems |

## Gotchas

- [[Virtual uniformSize caches first item dimension to avoid repeated DOM reads]] — `uniformSize: true` (default) assumes all items are the same size; disable for variable-size lists
- [[Virtual scrollToIndex resets container position before jumping to break accumulated animation offset]] — accumulated animation drift must be cleared before jumping; use the built-in `scrollToIndex` rather than manually setting `selected`
- [[VirtualGrid corrects selected index after re-slicing to keep focus on the right item]] — after a row change causes the slice to shift, VirtualGrid applies an index correction; user `onSelectedChanged` handlers chained before the internal one will see the pre-correction index
- [[Virtual component uses adaptive animation duration to handle rapid navigation]] — animation duration shortens automatically during fast navigation to prevent queuing buildup
- [[Lazy uses s.Index instead of s.For so items are stable by position not identity]] — LazyRow/Column children receive `index: number` (not reactive), unlike Virtual where both are accessors

### Measuring Performance

Use [[FPSCounter component displays real-time Lightning renderer performance metrics as a debug overlay]] during development to measure the impact of optimizations — it shows FPS, memory, texture counts, and draw calls in a live overlay.

## Cross-Domain Connections

Performance primitives integrate with [[styling guide]] and [[components guide]]:
- [[flex layout uses Float32Array typed arrays to cache child sizes for performance]] — flex layout itself is a performance-sensitive operation for large grids
- [[Virtual component uses adaptive animation duration to handle rapid navigation]] — connects performance to the animation system in the styling domain
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — the base containers that Virtual wraps
- [[createTag renders JSX off-screen as a reusable GPU texture for efficient repeated stamping]] — pairs well with Virtual lists where the same badge appears on hundreds of items
- [[createSpriteMap slices a sprite sheet into named Lightning SubTexture instances]] — GPU texture sharing for icon sets in large lists

## Open Questions
- How does KeepAlive integrate with Preserve at the routing layer?
- Does `eagerLoad: true` in Lazy interact with the task queue backpressure?
