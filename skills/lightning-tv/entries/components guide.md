---
description: Primitive components — Row, Column, Grid, Virtual, VirtualGrid, Image, Lazy, FadeInOut, Marquee, and more
type: moc
---

# components guide

The framework ships with 25+ reusable primitive components for building TV interfaces. Layout components (Row, Column, Grid), virtual scrolling, lazy loading, image handling, animation primitives, and state preservation.

## Core Concepts

### Layout Navigation Components
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — flex-row container with left/right navigation, auto-scroll, and 30px built-in gap; accepts full NavigableProps
- [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — flex-column container with up/down navigation; 300ms transitions vs Row's 180ms
- [[Grid component uses signal-driven focus and absolute positioning for 2D navigation]] — 2D grid using a render-prop children pattern, absolute item coordinates, and reactive signal-driven focus
- [[Grid uses Index not For so items are keyed by position and reordering updates in place]] — items are stable by position; reordering updates in place without remounting

### Image & Media
- [[Image component shows placeholder while src loads and fallback on error]] — extends NodeProps with `src`, `placeholder`, and `fallback`; handles both DOM and Lightning renderer paths; defaults color to white
- [[Image component uses dual signal system of src and texture in Lightning renderer mode]] — gotcha: both `src` (string) and `texture` (object) signals run in parallel; `texture` takes precedence once loaded; `getTextureData()` resolves even after failure so the `resp.data` guard is critical
- [[createBlurredImage reactive hook produces blurred data URLs using CPU-side Gaussian convolution]] — SolidJS Resource that re-blurs when URL changes; CSS filter when available, separable Gaussian fallback; `resolution` param scales canvas for speed

### Animation
- [[FadeInOut component blocks node destruction to play exit animation before removal]] — wraps `<Show keyed>` + `<View>` with alpha transitions; `onDestroy` returns a Promise blocking node removal; sets `rtt=true` before exit to prevent GPU glitches
- [[Marquee component scrolls overflowing text using two synchronized looping animations]] — two `<text>` nodes with `loop:true` animations offset by `textWidth+scrollGap`; only activates when `marquee=true` and text overflows; defers dual-node render until first focus

### State Preservation
- [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]] — general-purpose preservation via `preserve` prop + module-level Map; use when unmounting would lose expensive state

### UI Utilities
- [[borderBox directive injects a focus ring child into Lightning nodes using onFocusChanged]] — `use:borderBox` Solid directive; creates a border child on focus and destroys it on blur; only usable on native `view`/`text` elements
- [[Portal component mounts children at a target Lightning node found by ID from rootNode]] — renders children outside current tree position at `rootNode` or a named node; for overlays, toasts, modals
- [[Lightning Suspense keeps children in a hidden view to prevent node destruction during loading]] — critical: use this instead of `solid-js` Suspense to avoid destroying GPU nodes during data loading

### Developer Tools
- [[FPSCounter component displays real-time Lightning renderer performance metrics as a debug overlay]] — FPS, memory, texture counts, quads, draw calls; requires `setupFPS(root)` call; fixed-position at 1920px right edge

### Performance Primitives
- [[createTag renders JSX off-screen as a reusable GPU texture for efficient repeated stamping]] — renders JSX once to a GPU texture via RTT; returns a lightweight stamp component; use for badges/labels appearing many times in lists
- [[createSpriteMap slices a sprite sheet into named Lightning SubTexture instances]] — uploads sprite sheet once, creates named SubTexture descriptors; synchronous; efficient for many icons

### Composition Utilities
- [[chainFunctions composes event handlers where returning true stops the chain]] — standard utility across all primitives for composing user and internal handlers; `chainRefs` variant for ref forwarding

### Virtualized List Components

- [[VirtualRow and VirtualColumn render a sliding window over large item arrays]] — window-based virtualization for 1D lists; accepts full `each` array, renders only `displaySize + bufferSize` items at any time
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — 2D windowed grid with column-aligned slicing; `columns` prop required, `rows` defaults to 1
- [[VirtualRow and VirtualColumn track absolute cursor and relative selected as separate signals]] — internal design detail: `cursor` = absolute index, `selected` = slice position; must sync on every nav event
- [[Virtual wrap mode uses modular indexing to build a circular slice from any start position]] — `wrap: true` enables circular navigation using `mod` arithmetic and a -1 item container offset

### Lazy Loading Components

- [[LazyRow and LazyColumn incrementally render items one per frame until upCount is reached]] — deferred rendering; starts interactive immediately, loads remaining items at ~60fps in the background
- [[Lazy uses s.Index instead of s.For so items are stable by position not identity]] — `<Index>` keying means appended items don't cause existing items to remount; children receive non-reactive `index: number`
- [[lazy function loads a component asynchronously and exposes a preload method for eager fetching]] — code-splitting for screen components; `.preload()` enables route prefetching

### State Preservation Components

- [[Visible component toggles ElementNode hidden instead of destroying and recreating children]] — within-page show/hide without remount; children created once, toggled via `ElementNode.hidden`
- [[Preserve component marks an ElementNode as preserve=true to survive route transitions]] — renderer-level node preservation; `onRender`/`onRemove` lifecycle hooks toggle visibility across route changes

### Data Primitives

- [[createInfiniteItems accumulates paginated data into a single reactive array]] — pagination primitive pairing with Virtual's `onEndReached`; accumulates pages and auto-detects end

## Patterns

### Blurred placeholder while loading full image
```tsx
const blurred = createBlurredImage(() => props.src, { radius: 15, resolution: 0.5 });

<Image src={props.src} placeholder={blurred()} width={300} height={180} />
```

### Scrolling text on focus
```tsx
<Marquee width={400} marquee={inFocus()} speed={200} delay={1000}>
  {props.title}
</Marquee>
```

### Focus ring directive
```tsx
<view use:borderBox={{ borderSpace: 8, border: { color: 0x00bfff, width: 3 } }}>
  <ListItem />
</view>
```

### Conditional animated visibility
```tsx
<FadeInOut when={isMenuOpen()} transition={{ duration: 300 }}>
  <SideMenu />
</FadeInOut>
```

### RTT badge stamping for repeated icons
```tsx
const Badge = createTag(<view color={0xffd700ff} width={60} height={24}><text>PRE</text></view>);
// Use in many list items, clean up when done:
onCleanup(() => Badge.destroy());
```

### Building an Infinite-Scroll List

1. Use [[createInfiniteItems accumulates paginated data into a single reactive array]] to manage fetching
2. Pass `items()` to `VirtualRow`/`VirtualColumn`
3. Configure `onEndReachedThreshold` and `onEndReached` (see [[Virtual onEndReached fires when cursor moves within threshold of the last item]])

### Choosing Between VirtualRow, LazyRow, and a Plain Row

| Situation | Component |
|-----------|-----------|
| 500+ items, memory is a concern | VirtualRow/VirtualColumn |
| Slow to mount, navigation arrives first | LazyRow/LazyColumn |
| < ~50 items that mount quickly | Plain Row/Column |
| 2D grid layout | VirtualGrid |

### Conditional Visibility

Use [[Visible component toggles ElementNode hidden instead of destroying and recreating children]] for panels that toggle often. Reserve `<Show>` for components that should release resources when hidden.

## Gotchas

- [[Row and Column use @once annotations so handler props cannot be updated after mount]] — all handler/behavioral props on Row/Column are frozen at render via `/* @once */`; dynamic prop updates are silently ignored
- [[Row onLayout only chains scrollRow when selected prop is non-zero at mount]] — `selected={0}` (or unset) never triggers an initial scroll; pass a non-zero `selected` at mount time for initial offset
- [[Grid onSelectedChanged has a different signature than NavigableProps onSelectedChanged]] — `(index, grid, elm?)` vs Row/Column's `(index, elm, child, lastIdx?)`; incompatible if handlers are shared
- [[Grid scrollToIndex focuses the grid itself before the child and uses queueMicrotask for the child focus]] — side effect: calling scrollToIndex on an unfocused Grid steals focus to the grid
- [[Lightning Suspense keeps children in a hidden view to prevent node destruction during loading]] — never use `solid-js` Suspense directly in Lightning TV; always import from `@lightningtv/solid/primitives`
- [[Image component uses dual signal system of src and texture in Lightning renderer mode]] — `getTextureData()` resolves even after failure; missing the `resp.data` guard would set a failed texture overriding the fallback
- [[createTag renders JSX off-screen as a reusable GPU texture for efficient repeated stamping]] — 1ms timeout required after `rtt=true` before capturing texture; must call `.destroy()` to avoid memory leaks
- [[FPSCounter component displays real-time Lightning renderer performance metrics as a debug overlay]] — all instances share module-level signals; hardcoded 1920px position must be overridden for other screen sizes
- [[borderBox directive injects a focus ring child into Lightning nodes using onFocusChanged]] — directive limitation: cannot be applied to custom component roots, only native `view`/`text` elements
- [[Virtual uniformSize caches first item dimension to avoid repeated DOM reads]] — default `uniformSize: true` assumes uniform item sizes; set `false` for variable-size lists
- [[selectedNode getter mutates selected index as a side effect]] — reading el.selectedNode on a Row or Column scans forward past non-element nodes and updates this.selected; side effects on what looks like a read
- [[VirtualGrid corrects selected index after re-slicing to keep focus on the right item]] — slice index correction is internal; chained `onSelectedChanged` handlers see the pre-correction index
- [[Virtual component uses adaptive animation duration to handle rapid navigation]] — animation shortens automatically during fast nav; stop the old animation before starting a new one
- [[Virtual scrollToIndex resets container position before jumping to break accumulated animation offset]] — call built-in `scrollToIndex`, not manual `selected`; it clears animation drift first
- [[Lazy delay prop debounces item loading during rapid navigation]] — `delay` debounces loading; rapid nav cancels pending timeout and loads synchronously as fallback

## Cross-Domain Connections

Components depend on the [[styling guide]] and [[performance guide]]:
- Flex layout (see [[flex layout engine positions children using justifyContent along the main axis]]) is the foundation for Row, Column, and VirtualGrid
- State variants (see [[dollar-prefix state keys in NodeStyles apply style variants based on active states]]) drive focus styling in all list components
- The task queue (see [[scheduleTask enables deferred background work with high and low priority]]) powers LazyRow/Column eagerLoad mode
- Animatable transitions (see [[animatable number properties route through transition system before reaching the renderer]]) underpin Row/Column scrolling, FadeInOut, and Marquee

## Open Questions
- How does KeepAlive interact with focus when inside a Row/Column that tracks scroll position?
- Does `Portal` work correctly inside a `KeepAlive` preserved route?
- What other primitives are exported from `@lightningtv/solid/primitives` beyond what is documented here?
