---
description: When Proxy is unavailable or forceProxy is true, params and query are backed by per-property createMemo calls on a static key set
type: gotcha
module: routing
created: 2026-04-07
---

# HashRouter uses proxy-free memo fallback for TV environments without Proxy support

Some embedded TV browsers do not support the JavaScript `Proxy` object. The `HashRouter` detects this via `SUPPORTS_PROXY = typeof Proxy === 'function'` and falls back to `createMemoWithoutProxy`.

**The problem**: `@solidjs/router` normally uses a `Proxy` to lazily intercept property access on `params` and `query` objects, creating memos on demand. Without `Proxy`, all needed keys must be known in advance.

**The fallback**: `createMemoWithoutProxy<T>(fn, allKeys?)` creates a plain object where each property is backed by its own `createMemo(() => fn()[key])`. Keys are either passed explicitly via `allKeys` or inferred from `Object.keys(fn())` at creation time.

**Declaring query params**: if your route uses query parameters and you're in a Proxy-free environment, pass them via the `queryParams` prop:
```tsx
<HashRouter queryParams={['page', 'filter', 'sort']}>
```

**Dynamic route params**: `collectDynamicParams(branches)` scans route patterns (`:param` syntax) and returns the unique param names — used to pre-build memos for each without Proxy.

This gotcha is silent: if a param is accessed that wasn't in the initial key set in a Proxy-free environment, it returns undefined rather than a reactive value. Always declare params explicitly when targeting Proxy-limited devices.

---

Source: [[primitives-router]]
Domains:
- [[routing guide]]
