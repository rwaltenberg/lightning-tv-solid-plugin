---
description: Routing — hash-based router using @solidjs/router, KeepAlive for route state preservation, before-leave hooks
type: moc
---

# routing guide

Routing uses `@solidjs/router` with a hash-based parser adapted for Lightning TV apps. Routes use `/#/path` URLs via `window.location.hash`. `KeepAlive` preserves both SolidJS reactive scope and Lightning renderer GPU nodes across route changes. The router supports nested routes, before-leave hooks, and proxy-free operation for embedded TV browsers.

## Core Concepts

- [[HashRouter wraps solidjs router with hash-based URL parsing for TV environments]] — the primary router component; reads/writes `window.location.hash`, binds `hashchange` events, integrates before-leave guards
- [[HashRouter uses proxy-free memo fallback for TV environments without Proxy support]] — critical gotcha for embedded browsers; `createMemoWithoutProxy` pre-computes property accessors when `Proxy` is unavailable; use `queryParams` prop to declare params ahead of time
- [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]] — wraps children with `preserve` prop, stores them in a module-level Map keyed by id; returns same children on re-mount without re-render
- [[KeepAliveRoute restores focused element when returning to a preserved route]] — router-integrated KeepAlive that saves `activeElement()` on leave and walks the parent chain to restore it on return
- [[isAlive accessor from KeepAliveRoute lets components pause work when their route is not visible]] — components receive `isAlive: Accessor<boolean>` to gate timers, polling, and animations while the route is hidden

## Patterns

### Basic hash routing setup
```tsx
<HashRouter>
  <Route path="/" component={Home} />
  <Route path="/detail/:id" component={Detail} />
</HashRouter>
```

### Preserved routes with focus restoration
```tsx
<HashRouter>
  <KeepAliveRoute path="/browse" component={({ isAlive }) => <Browse isAlive={isAlive} />} />
  <Route path="/detail/:id" component={Detail} />
</HashRouter>
```

### Proxy-free TV environments
```tsx
<HashRouter queryParams={['page', 'filter']} forceProxy>
  ...
</HashRouter>
```

### Manually managing KeepAlive
```ts
import { removeKeepAlive, clearKeepAliveRoute } from '@lightningtv/solid/primitives';

// Remove a specific preserved route (calls destroy + dispose)
removeKeepAlive('browse-screen');

// Clear all preserved routes
clearKeepAliveRoute();
```

## Gotchas

- [[HashRouter uses proxy-free memo fallback for TV environments without Proxy support]] — silently returns `undefined` for undeclared params in proxy-free mode; always use `queryParams` prop when targeting embedded browsers
- [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]] — `shouldDispose` callback allows conditional teardown; if omitted, elements are preserved indefinitely until `removeKeepAlive` is called
- [[isAlive accessor from KeepAliveRoute lets components pause work when their route is not visible]] — without `isAlive` checks, background routes continue running effects, animations, and timers, wasting resources

## Open Questions

- What are the limits on how many routes can be preserved simultaneously before GPU memory pressure causes issues?
- Does KeepAlive interact with @solidjs/router's preload mechanism beyond what KeepAliveRoute wraps?
- How does the `shouldDispose` prop integrate with route parameter changes (e.g., same route, different id)?
