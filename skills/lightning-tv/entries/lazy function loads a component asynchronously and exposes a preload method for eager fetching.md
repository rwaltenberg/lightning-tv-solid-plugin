---
description: Custom `lazy()` wraps dynamic import, deduplicating the Promise and returning a component that renders null until resolved, with `.preload()` for route prefetching
type: api
module: primitives
created: 2026-04-07
---

The `lazy` function from `LazyImport.ts` enables code-splitting for screen components:

```ts
const MyScreen = lazy(() => import('./screens/MyScreen'));

// Optional: prefetch before navigation
MyScreen.preload();
```

## Signature

```ts
function lazy<T extends Component<any>>(
  fn: () => Promise<{ default: T }>,
): T & { preload: () => Promise<{ default: T }> }
```

## Behavior

- The returned value acts as a normal component: `<MyScreen title="Home" />`
- Before the import resolves, the component renders `null`
- After resolution, it renders normally
- Multiple calls to `fn()` are deduplicated via a shared `p` Promise variable
- `createResource` provides reactive integration — the component re-renders when the import resolves

## Preload Pattern

```ts
wrap.preload = () =>
  p || ((p = fn()).then(mod => (comp = () => mod.default)), p);
```

Calling `preload()` starts the import early (e.g., when the user hovers over a nav item or when a route becomes likely). When the component later renders, `comp` is already set and no waiting occurs.

## SSR/Hydration Path

When `sharedConfig.context` is set (server-side rendering):
1. Uses `createSignal` instead of `createResource`
2. Increments `sharedConfig.count` to block hydration until resolved
3. Temporarily restores `sharedConfig.context` during render for hydration matching

For most Lightning TV use cases (client-only), the SSR path is irrelevant.

## When to Use

Use `lazy` to split route screens into separate JS bundles. Combined with `preload()` calls in routing logic, this enables fast perceived navigation: the next screen's code is fetched during the current screen's idle time.

---

Source: [[primitives-LazyImport]]
Domains:
- [[performance guide]]
- [[components guide]]
- [[routing guide]]
