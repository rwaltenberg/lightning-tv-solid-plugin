---
description: If any ElementNode in the current focus path has loading set to true, the Announcer skips the announcement entirely and re-announces when Announcer.refresh() is called after loading completes
type: pattern
module: primitives
created: 2026-04-07
---

# Announcer loading property defers speech until all elements in the focus path are loaded

The `loading` property on `ElementNode` acts as an announcement gate. When any element in the focus path is still loading, `onFocusChangeCore` passes an empty array to `onFocusChange` instead of the actual focus path:

```typescript
const loaded = focusPath.every((elm) => !elm.loading);
if (!loaded && Announcer.onFocusChange) {
  Announcer.onFocusChange([]); // triggers with empty path → no speech
  return;
}
```

## Why This Matters

TV applications often render list rows or hero items asynchronously. When a user navigates to a row that is still fetching its data, announcing "undefined" or an empty label would be confusing. The `loading` flag prevents premature announcements.

## Correct Usage Pattern

```typescript
// Set loading while fetching
myElement.loading = true;

// Fetch content...
const data = await fetchContent();
myElement.announce = data.title;
myElement.loading = false;

// Re-announce now that content is ready
Announcer.refresh();
```

`Announcer.refresh()` clears `prevFocusPath` (treating all current elements as "new") and re-triggers `onFocusChange` with the current focus path. Since `loading` is now `false`, the full announcement proceeds.

## Relationship to `prevFocusPath`

When `loading` is true, `prevFocusPath` is NOT updated. This means the deferred announcement will include ALL elements in the focus path (not just newly focused ones), since none of them were in the acknowledged `prevFocusPath`.

Since [[Announcer singleton announces newly focused elements by diffing the focus path against previous state]], the loading gate interacts cleanly with the diff mechanism to ensure no partial announcements escape.

This is particularly relevant after route transitions — when returning to a [[KeepAliveRoute restores focused element when returning to a preserved route]] that was preserved but needs fresh data, set `loading = true` on content elements before fetching and call `Announcer.refresh()` once data is ready.

---

Source: [[primitives-announcer]]
Domains:
- [[accessibility guide]]
- [[routing guide]]
