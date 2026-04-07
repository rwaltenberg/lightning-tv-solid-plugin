---
description: The calculateFlex function is selected at module load time from VITE_USE_NEW_FLEX env var; the new engine adds flexShrink, flexBasis, flexCrossBoundary, and individual padding sides
type: architecture
module: core
created: 2026-04-07
---

# VITE_USE_NEW_FLEX selects between old and new flex layout engines

At the top of `elementNode.ts`, the flex calculation function is selected once at module initialization:

```typescript
const calculateFlex = (import.meta as any).env?.VITE_USE_NEW_FLEX
  ? calculateFlexNew
  : calculateFlexOld;
```

`calculateFlexOld` is imported from `./flex.js` and `calculateFlexNew` from `./flexLayout.js`.

## Feature differences

The new flex engine (`calculateFlexNew`) adds support for properties that are marked "Only available in NEW flex layout" in the `ElementNode` interface:

- **`flexShrink`** — how much an item can shrink (defaults to 0 in old engine, matching legacy behavior)
- **`flexBasis`** — default size before remaining space is distributed (number or string)
- **`flexCrossBoundary`** — controls cross-axis sizing (`'fixed'`; default is `contain`)
- **`paddingTop`**, **`paddingRight`**, **`paddingBottom`**, **`paddingLeft`** — individual padding sides (new engine only)

The old engine only supports the `padding` array shorthand.

**Gotcha:** Since the selection is at module load time (not runtime), you cannot switch engines without a Vite rebuild. Components using new-engine-only props (`flexShrink`, `flexBasis`, individual padding) will silently have no effect when running with the old engine.

---

Related Entries:
- [[flexLayout.ts adds flexBasis flexShrink and per-side padding over the original flex implementation]] — detailed comparison of what flexLayout.ts adds over flex.ts
- [[flexShrink distributes overflow reduction proportionally among children]] — one of the key new-engine-only features

Source: [[core-elementNode]]
Domains:
- [[core guide]]
