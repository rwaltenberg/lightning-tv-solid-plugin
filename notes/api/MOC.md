# API Reference -- Primitives & Utilities

Full props, signatures, defaults, and behavioral notes for every component and utility.

## Core APIs

- [element-node](element-node.md) -- ElementNode: all methods, properties, proxy behavior, animation
- [config](config.md) -- Config object: all fields, defaults, compile-time globals
- [states-class](states-class.md) -- States class: add, remove, toggle, merge, has, is

## Focus & Input APIs

- [use-focus-manager](use-focus-manager.md) -- useFocusManager: KeyMap, KeyHoldOptions, owner context
- [focus-stack-provider](focus-stack-provider.md) -- FocusStackProvider, useFocusStack: store/restore/clear
- [handle-navigation](handle-navigation.md) -- handleNavigation(), moveSelection(), navigableHandleNavigation
- [forward-focus-handlers](forward-focus-handlers.md) -- navigableForwardFocus, spatialForwardFocus signatures

## Navigation Primitives

- [row](row.md) -- Row: RowProps, default styles, scroll, transitions, forwardFocus wiring
- [column](column.md) -- Column: ColumnProps, vertical equivalent of Row
- [grid](grid.md) -- Grid: GridProps, GridItemProps, absolute positioning, looping nav
- [virtual-row](virtual-row.md) -- VirtualRow: VirtualProps, Accessor pattern, windowing, onEndReached
- [virtual-column](virtual-column.md) -- VirtualColumn: vertical virtual list
- [virtual-grid](virtual-grid.md) -- VirtualGrid: VirtualGridProps, columns, rows, buffer, flexWrap
- [lazy-row](lazy-row.md) -- LazyRow: LazyProps, upCount, progressive render, eagerLoad
- [lazy-column](lazy-column.md) -- LazyColumn: vertical lazy list

## Display Primitives

- [image](image.md) -- Image: placeholder/fallback lifecycle, DOM vs Lightning paths
- [fade-in-out](fade-in-out.md) -- FadeInOut: rtt trick for exit animation, ALPHA_NONE/ALPHA_FULL
- [marquee](marquee.md) -- Marquee/MarqueeText: clipWidth, speed, delay, scrollGap, clipping
- [visible](visible.md) -- Visible: vs Show, hidden toggling, createRoot, no node destruction
- [keep-alive](keep-alive.md) -- KeepAlive: keepAliveElements Map, KeepAliveRoute, route integration
- [portal](portal.md) -- Portal: mount prop, rootNode.searchChildrenById
- [suspense-primitive](suspense-primitive.md) -- Suspense: focus tree preservation, hidden view wrapper
- [fps-counter](fps-counter.md) -- FPSCounter: setupFPS, singleton, renderer event subscriptions

## Utility APIs

- [use-announcer](use-announcer.md) -- useAnnouncer: announce/announceContext/title props, speech modes
- [use-mouse](use-mouse.md) -- useMouse: hover BFS, click dispatch, scroll wheel mapping
- [use-hold](use-hold.md) -- useHold: threshold, onHold, onRelease, onEnter
- [chain-functions](chain-functions.md) -- chainFunctions/chainRefs: true stops chain, ref forwarding
- [create-infinite-items](create-infinite-items.md) -- createInfiniteItems: fetcher, page, setPage, end signal
- [scrolling](scrolling.md) -- scrollRow/scrollColumn: all 6 scroll modes with behavior details
