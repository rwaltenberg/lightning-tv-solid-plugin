---
description: Provides hash-based client-side routing via @solidjs/router primitives; reads/writes window.location.hash and listens to hashchange events
type: api
module: routing
created: 2026-04-07
---

# HashRouter wraps solidjs router with hash-based URL parsing for TV environments

`HashRouter` is the primary router component for Lightning TV applications. It adapts `@solidjs/router`'s `createRouter` primitive to use hash-based URLs (`/#/path`) rather than History API paths.

```tsx
import { HashRouter } from '@lightningtv/solid/primitives';

function App() {
  return (
    <HashRouter>
      <Route path="/" component={Home} />
      <Route path="/detail/:id" component={Detail} />
    </HashRouter>
  );
}
```

**Props** extend `BaseRouterProps` with:
- `forceProxy?: boolean` — forces proxy-free memo mode even when `Proxy` is available
- `queryParams?: string[]` — pre-declares query param names for proxy-free environments
- `explicitLinks?: boolean`, `preload?: boolean`, `actionBase?: string`

**Internal routing mechanics**:
- `get`: reads `window.location.hash.slice(1)`
- `set`: pushes/replaces via `window.history.pushState` with `#` prefix
- `init`: binds `hashchange` event via `bindEvent` for reactive updates
- Before-leave guard integrated via `createBeforeLeave` and `notifyIfNotBlocked`

The `hashParser` utility handles both route links (`/#/foo` → `/foo`) and in-page anchor links (`/#foo` → appended as fragment to current path).

`HashRouter` is typically used alongside [[KeepAlive preserves Lightning renderer nodes and reactive scope across route navigation]] via `KeepAliveRoute` — the router drives URL changes while KeepAlive ensures GPU nodes and SolidJS reactive scopes are not destroyed on route exit.

---

Source: [[primitives-router]]
Domains:
- [[routing guide]]
