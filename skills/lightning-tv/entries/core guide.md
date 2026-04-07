---
description: Core framework internals — ElementNode class, renderer initialization, global config, and task scheduling
type: moc
---

# core guide

The foundation of Lightning TV/Solid. ElementNode is the primary abstraction — a DOM-like node for WebGL rendering. The renderer bridges SolidJS reactivity with LightningJS 3's WebGL engine.

## Core Concepts

### ElementNode — Core Class

- [[ElementNode is the primary abstraction for all renderable elements in Lightning TV Solid]] — every visible element is an instance of this class; extends Object not HTMLElement; handles both text and element node types in one class
- [[ElementNode lng property holds a plain object before render and the live renderer node after]] — the two-phase design that lets SolidJS accumulate props safely before tree attachment
- [[ElementNode render method controls the two-phase lifecycle from accumulation to live rendering]] — when and how props become a live renderer node; handles positioning, sizing defaults, font settings, and recursive child rendering
- [[animatable number properties route through transition system before reaching the renderer]] — how LightningRendererNumberProps and _sendToLightningAnimatable() enable declarative transitions on x, y, w, h, alpha, color, and others
- [[signal-to-GPU pipeline routes property changes through proxy setters to the renderer]] — Object.defineProperty setters on LightningRendererNumberProps intercept every write and route through _sendToLightningAnimatable before reaching this.lng
- [[layout queue defers flex recalculation and processes children before parents]] — the Set-based deduplication and microtask scheduling that batches all layout work; reverse-order traversal
- [[ElementNode updateLayout method runs flex and onLayout for flex containers]] — the actual flex calculation step; propagates dimension changes up the tree; handles flexGrow second pass
- [[ElementNode animate method requires rendered state and returns a chainable IAnimationController]] — direct imperative animation; asserts rendered; fallback settings chain
- [[ElementNode chain and start methods enable sequenced animation queuing]] — how to build multi-step sequential animations with queue reset behavior when called mid-animation
- [[onDestroy can return a promise to delay node destruction for exit animations]] — pattern for coordinating exit animations with node cleanup via promise return
- [[setFocus defers focus assignment via microtask to allow children to render first]] — the nextActiveElement + focusQueued deduplication pattern; forwardFocus handling; pre-render autofocus path
- [[ElementNode emit method bubbles custom events up the parent chain]] — onX handler discovery, bubbling semantics, return-true cancellation
- [[states setter merges new states into existing States object and triggers immediate style application]] — how state mutations trigger visual updates; lazy States creation
- [[_stateChanged applies dollar-prefix style variants with undo tracking and state order priority]] — the full state-to-style pipeline; undo tracking; transition applied before other styles
- [[stateOrder controls specificity when multiple states are active simultaneously]] — how Config.stateOrder and node-level stateOrder resolve conflicts when multiple states are active
- [[forwardStates propagates parent state array to all children on every state change]] — group state propagation; one-level deep; merge semantics
- [[style setter only applies properties not already set by JSX props]] — JSX prop priority over style object; Config.lockStyles; dev double-set warning
- [[effects setter builds shader type by composing feature flags and converts before render]] — how rounded, border, shadow combine into shader type names; SHADERS_ENABLED guard; two-phase conversion
- [[fontWeight is encoded as a font family name suffix using Config fontWeightAlias]] — the naming convention for font weights; alias lookup; deduplication guard
- [[right and bottom props compute position from parent edges by setting mount to 1]] — edge-relative positioning computed once at render time; center/centerX/centerY handling
- [[element node NaN width and height default to parent dimensions minus position offset]] — fill-parent behavior; flexGrow gets w=0 instead; texture nodes exempt
- [[width and height getters return max constraint not actual dimension when maxWidth or maxHeight is set]] — the maxWidth || w pattern; implications for layout debugging
- [[text node contain prop maps to maxWidth and maxHeight during render]] — contain='width' vs 'both'; maxLines default; textAlign requires contain; post-render layout trigger
- [[onEvent prop binds renderer lifecycle events to the ElementNode instance]] — loaded, failed, freed, inBounds, outOfBounds, inViewport, outOfViewport; handler binding during render()
- [[insertChild mirrors DOM insertBefore and removes node from prior parent enabling node moves]] — automatic parent removal; _hasRenderedChildren flag for rendered-child-into-unrendered-container case
- [[element nodes default color to transparent unless src or explicit color is set]] — the 0x00000000 default; src setter auto-applies white post-render; rtt exception
- [[VITE_USE_NEW_FLEX selects between old and new flex layout engines]] — build-time engine selection; new engine features (flexShrink, flexBasis, flexCrossBoundary, individual padding)

### Node Type System
- [[node type discriminators allow the flex engine to skip text nodes]] — NodeType enum with three values (element, textNode, text) used by isTextNode() and isElementText() for runtime discrimination
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — the full prop interface merging renderer props, focus callbacks, key handlers, and ElementNode properties; NodeStyles adds dollar-prefix state variants
- [[TextProps and TextStyles are the JSX prop types for text nodes]] — parallel type for Text elements; deliberately excludes layout, focus-routing, and visual props not applicable to text
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — $-prefixed style keys override properties when matching state is active
- [[Effects interface provides eight shader effects applicable to element nodes]] — composable visual effects: gradients, shadow, holePunch, rounded, borders
- [[node lifecycle events fire on loaded failed freed and bounds changes]] — OnEvent type for loaded, failed, freed, inBounds/outOfBounds, inViewport/outOfViewport

### Configuration and Initialization
- [[Config singleton holds all runtime Lightning configuration]] — the mutable global object controlling debug, animations, fonts, focus, shaders, and state ordering; must be configured before importing Lightning modules
- [[startLightningRenderer initializes the global renderer singleton]] — creates either WebGL RendererMain or DOMRendererMain and assigns it to the module-level renderer export
- [[loadFonts dispatches font loading based on renderer mode and font type]] — routes SDF fonts to the WebGL stage and web fonts to canvas/DOM; silently skips mismatched fonts
- [[States class extends Array to manage dollar-prefixed state strings]] — the class behind ElementNode.states; extends native Array with add/remove/toggle/merge and an onChange callback
- [[flattenStyles merges style arrays with first-value-wins priority]] — recursive style merger used throughout the framework; earlier entries in arrays take priority, 0-values preserved
- [[getElementScreenRect calculates absolute screen coordinates accounting for scale and DPR]] — walks parent chain accumulating position and scale offsets, applies deviceLogicalPixelRatio
- [[log utility enables per-node debug logging without global debug flag]] — conditional logger that respects both Config.debug and per-node debug properties, zero cost in production
- [[logRenderTree generates reproducible renderer code for debug isolation]] — outputs self-contained JS to recreate a node tree against the renderer API, useful for isolating rendering bugs
- [[isFunc and isFunction are intentionally different type guards]] — two distinct function checks: instanceof Function vs typeof function, not interchangeable

### SolidJS Integration and Render Bridge
- [[solidOpts bridges SolidJS reconciler to the Lightning ElementNode tree]] — the 9-method nodeOpts object that adapts SolidJS's universal renderer to Lightning node operations
- [[node move uses counter-balanced delete queue to avoid premature destruction]] — remove pushes -1, insert pushes +1; only net-negative nodes at microtask flush are destroyed, enabling atomic moves
- [[preserve property prevents node destruction by holding the delete counter at zero]] — setting preserve=true fixes _queueDelete=0, permanently exempting the node from the destruction path
- [[activeElement is a module-level SolidJS signal tracking the currently focused ElementNode]] — injected into Config.setActiveElement so the focus manager drives Solid reactivity without circular imports
- [[View and Text components avoid JSX to prevent circular dependency issues]] — createElement('node'/'text') + spread(el, props, false) avoids circular deps in playground environments

### Task Queue
- [[scheduleTask enables deferred background work with high and low priority]] — public API for background tasks; high pushes to front; runs one per setTimeout at Config.taskDelay (default 50ms)
- [[task queue suspends on focus change and resumes when renderer is idle]] — reactive effect on activeElement() disables queue immediately on keypress; renderer 'idle' event re-enables it

### Utilities
- [[hexColor converts string color formats to RGBA numbers for the Lightning renderer]] — handles #RRGGBB, #RRGGBBAA, 0x-prefixed, bare 6-char hex, and passthrough integers
- [[combineStyles and combineStylesMemo merge two style objects with first-object-wins priority]] — combineStyles is plain spread; combineStylesMemo returns reactive Accessor with createMemo only when both defined
- [[getWebglSupportedVersions detects available WebGL context versions by probing canvas contexts]] — cached canvas probe; supportsWebGL/supportsWebGL2/supportsOnlyWebGL2 predicate wrappers

### DOM Renderer
- [[DOMNode renders Lightning visual properties as computed inline CSS on a div element]] — single setAttribute('style') per update; divBg layer for masks/images; divBorder layer for borders with masks
- [[DOM renderer defers image loading until node enters bounds using renderState tracking]] — lazyImagePendingSrc holds URL; img.src only set when renderState is 4 (inBounds) or 8 (inViewport)
- [[DOM renderer AnimationController interpolates numeric props using requestAnimationFrame]] — rAF-based loop; per-channel color interpolation; delay/repeat/loop/easing; waitUntilStopped() promise
- [[DOMText measures its own dimensions after render to feed back into flex layout]] — getBoundingClientRect() / DPR / parent scale; deferred via setTimeout after fonts load; emits 'loaded' on change

## Patterns

- [[Config.fontWeightAlias maps named weights to numeric values]] — extend this map when using non-standard font weight names
- [[Config.focusStateKey controls which state string marks focused elements]] — isFocused() uses this key; change it to rename the focus state across your app

## Gotchas

- [[element nodes default color to transparent unless src or explicit color is set]] — elements without src or explicit color are invisible (0x00000000); src setter auto-applies white post-render
- [[color 0x00000001 is the safe workaround for nodes that will later receive a dynamic src]] — near-transparent color blocks the src setter's auto-white behavior while staying visually invisible before load
- [[selectedNode getter mutates selected index as a side effect]] — reading selectedNode advances this.selected to skip non-element nodes; not a pure read; can shift selected index unexpectedly
- [[width and height getters return max constraint not actual dimension when maxWidth or maxHeight is set]] — node.width ≠ node.w when maxWidth is set; check both separately when debugging layout
- [[style setter only applies properties not already set by JSX props]] — style object does NOT override JSX props; only fills in missing values; double-set warns in dev
- [[text node contain prop maps to maxWidth and maxHeight during render]] — textAlign silently ignores alignment without contain; both w and h needed for contain='both'
- [[fontWeight is encoded as a font family name suffix using Config fontWeightAlias]] — wrong font naming convention = silent fallback font; alias must match registered font name
- [[right and bottom props compute position from parent edges by setting mount to 1]] — computed once at render; not reactive to subsequent parent size changes
- [[dom renderer requires both build-time flag and runtime Config flag to activate]] — LIGHTNING_DOM_RENDERING AND Config.domRendererEnabled must both be true; one alone silently falls back to WebGL
- [[LIGHTNING_DISABLE_SHADERS global flag disables all shader effects at build time]] — must be set before module load; SHADERS_ENABLED is derived at import time and cannot change at runtime
- [[States has method tolerates dollar prefix in both directions]] — has('focus') matches '$focus' as a backward-compatibility shim; marked as temporary in source, use full prefix always
- [[States merge method replaces or patches state collection depending on input type]] — array/string input clears all states first; object input patches selectively; neither fires onChange
- [[loadFonts dispatches font loading based on renderer mode and font type]] — mismatched font formats (SDF in canvas mode, web font in WebGL mode) are silently ignored

## Open Questions
- How does ElementNode lifecycle interact with SolidJS reactivity at the signal/effect level?
- What is the full behavior of Config.lockStyles — what exactly gets frozen beyond the style setter?
- How does _calcWidth/_calcHeight affect flex growth calculations in flexLayout.ts?
- What is the full renderer.stage.reprocessUpdates contract vs queueMicrotask?
