---
description: Styling system — flex layout engine, custom states with dollar-prefix, transitions, and shader effects
type: moc
---

# styling guide

Lightning TV/Solid uses a CSS-like but WebGL-rendered styling system. Flex layout, state-based style variants ($focus, $hover), animated transitions, and shader effects (rounded corners, shadows, gradients, borders).

## Core Concepts

### Flex Layout
- [[flex layout engine positions children using justifyContent along the main axis]] — six justifyContent values distribute children along the main axis; flexStart is the default and triggers container auto-resize
- [[flexDirection controls main axis orientation and supports row-reverse and column-reverse]] — four direction values; reverse variants and RTL both call the same index-reversal code path
- [[alignItems and alignSelf position children on the cross axis]] — cross-axis alignment with three values; requires containerCrossSize to be non-zero
- [[flex container resizes to content by default unless flexBoundary is fixed]] — containers shrink/grow to fit children by default; opt out with flexBoundary='fixed'
- [[flexGrow distributes positive available space proportionally among children]] — proportional expansion into remaining space; auto-sets flexBoundary fixed; uses _containsFlexGrow guard
- [[flexShrink distributes overflow reduction proportionally among children]] — weighted shrink when children overflow; only available in flexLayout.ts
- [[flexWrap wraps children to new rows when they overflow the main axis]] — wrap and wrap-reverse modes; cross-dimension expands to fit all rows
- [[flexOrder reorders children visually without changing DOM order]] — sort by ascending flexOrder value; happens before direction reversal
- [[flexItem false excludes a child from flex layout calculations]] — opt child out of flex entirely; position/size unchanged
- [[flex engine auto-calculates row container height from tallest child via _calcHeight flag]] — internal flag for row containers to match tallest child height
- [[flexLayout.ts adds flexBasis flexShrink and per-side padding over the original flex implementation]] — comparison of the two flex engine files; flexLayout.ts is the more complete implementation
- [[VITE_USE_NEW_FLEX selects between old and new flex layout engines]] — build-time env var selects which engine runs; new-engine-only props silently no-op with old engine
- [[flex layout uses Float32Array typed arrays to cache child sizes for performance]] — performance optimization for large grids

### State Variants
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — $focus, $hover, $disabled pattern using the NodeStyles index signature
- [[States class extends Array to manage dollar-prefixed state strings]] — manages the active set of dollar-prefixed state names on each element; drives style variant application ($focus, $hover, $disabled, etc.)
- [[Config.focusStateKey controls which state string marks focused elements]] — the key (default '$focus') added to element states on focus, used by isFocused() and style variant resolution
- [[Config.stateOrder controls style resolution priority when multiple states are active]] — explicit DollarString[] ordering that determines which state's styles win when multiple states are active simultaneously
- [[available WebGL shader types and their registration keys]] — eight built-in shaders: rounded, shadow, roundedWithBorder, roundedWithShadow, roundedWithBorderWithShadow, holePunch, radialGradient, linearGradient
- [[ShaderBorderProps extends Lightning BorderProps with gap and inset fields]] — Solid adds gap (space between border and element) and inset (draw inside vs outside) to the base renderer border API
- [[flattenStyles merges style arrays with first-value-wins priority]] — style composition utility; earlier styles win in array order, 0-values correctly preserved
- [[combineStyles and combineStylesMemo merge two style objects with first-object-wins priority]] — for the two-object case; combineStylesMemo is reactive (createMemo) only when both args are defined
- [[hexColor converts string color formats to RGBA numbers for the Lightning renderer]] — converts #RRGGBB, 0x-prefixed, and bare hex strings into 32-bit RGBA for use in color props

## Patterns

- [[shader radius array is automatically scaled to fit element dimensions]] — oversized radius values are silently scaled down proportionally; single number expands to [n,n,n,n], 2-value to [a,b,a,b]
- [[States merge method replaces or patches state collection depending on input type]] — use object form for selective state changes, array form to replace all states at once

## Gotchas

- [[flex engine returns false early when ElementText child has text but no explicit dimensions]] — mixing unsized Text nodes in a flex container silently prevents layout from running
- [[flexGrow is silently ignored when container has only one processed child]] — the flexGrow distribution block requires numProcessedChildren > 1; single-child containers see no expansion; use width={NaN} as workaround
- [[RTL direction combined with row-reverse produces only a single reversal not a double]] — flex.ts combines isReverse and RTL in one || condition; only one .reverse() call happens; combining them does not cancel out
- [[shader transitions fail silently when animationSettings is undefined]] — shaderAccessor only starts animation when animationSettings is truthy; transition=true and transition={border:true} both resolve to undefined and skip animation
- [[shaders are silently skipped when DOM renderer is active]] — all shader effects (rounded corners, shadows, gradients) produce no output in DOM mode; no error or warning
- [[RoundedWithBorderAndShadow shader does not support border-gap property]] — border-gap is in the TypeScript type but the underlying @lightningjs/renderer shader ignores it; documented TODO in source
- [[States has method tolerates dollar prefix in both directions]] — has('focus') matches '$focus', but this is a temporary compatibility shim; always use the full $ prefix
- [[LIGHTNING_DISABLE_SHADERS global flag disables all shader effects at build time]] — must be a global before module import; cannot be toggled at runtime

## Patterns

Use `flexBoundary="fixed"` when the container must maintain a fixed size regardless of child count. Leave it unset when the container should shrink-wrap its children.

When using `flexGrow`, note that `flexBoundary` is auto-set — you don't need to declare it explicitly.

For cross-axis alignment to work, the container must have an explicit `height` (for row containers) or `width` (for column containers), or use `_calcHeight` to auto-calculate from children.

## Cross-Domain Connections

Styling integrates tightly with the [[components guide]] and [[performance guide]]:
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] and [[Column component is a vertical navigable list with flexDirection column and 30px gap]] — built on flex layout; Row uses justifyContent:flexStart and Column adds flexDirection:column
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — depends on flexWrap to arrange items into rows
- [[FadeInOut component blocks node destruction to play exit animation before removal]] — uses alpha from the animatable properties system
- [[forwardStates propagates parent state array to all children on every state change]] — the bridge between focus state and $focus style variants in list items

## Open Questions
- How do state style variants compose and override within a single element's style object?
- How does lockStyles interact with style mutation after render?
