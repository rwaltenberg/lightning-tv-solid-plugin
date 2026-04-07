---
description: Modified SolidJS Suspense that renders children in a hidden view while loading rather than removing them, preserving GPU resources and focus state
type: gotcha
module: primitives
created: 2026-04-07
---

# Lightning Suspense keeps children in a hidden view to prevent node destruction during loading

The `Suspense` component from `@lightningtv/solid/primitives` is a Lightning-specific adaptation of SolidJS's `Suspense`. The standard SolidJS version removes children from the tree while loading, which in Lightning's renderer frees GPU memory. This component keeps children alive in a hidden view.

```tsx
import { Suspense } from '@lightningtv/solid/primitives';

const [data] = createResource(async () => fetchContent());

<Suspense fallback={<LoadingSpinner />}>
  <ContentView data={data()} />
</Suspense>
```

**Behavior during loading**:
- Top level: renders `props.fallback`
- Hidden view: renders children (invisible via `hidden` prop)
- `forwardFocus={0}` on the hidden view means focus passes through to children even while hidden

**Behavior when loaded**:
- Top level: renders children normally
- Hidden view: renders nothing (`null`)

**Why use this instead of standard Suspense**: standard `s.Suspense` destroys and recreates Lightning nodes on each load cycle. For complex views with many textures, this causes visible re-loading. This version preserves the node tree, so textures remain in GPU memory.

**Critical gotcha**: do NOT import `Suspense` from `solid-js` in Lightning TV applications. Always import from `@lightningtv/solid/primitives`. The standard version will work functionally but degrade performance and cause visual flicker on re-fetches.

---

Source: [[primitives-Suspense]]
Domains:
- [[components guide]]
- [[performance guide]]
