---
description: updateLayout() runs calculateFlex and calls onLayout on flex containers; propagates to parent if dimensions changed; handles flexGrow children with a second pass
type: api
module: core
created: 2026-04-07
---

# ElementNode updateLayout method runs flex and onLayout for flex containers

`updateLayout()` is the method called by the layout queue runner to perform actual flex calculations:

```typescript
updateLayout() {
  if (this.hasChildren) {
    // Skip flex containers with flexGrow if width is 0
    if (this.display === 'flex' && this.flexGrow && this.width === 0) {
      return;
    }

    const flexChanged = this.display === 'flex' && calculateFlex(this);
    layoutQueue.delete(this);  // remove self to prevent double-processing
    const onLayoutChanged = isFunc(this.onLayout) && this.onLayout.call(this, this);

    if ((flexChanged || onLayoutChanged) && this.parent) {
      addToLayoutQueue(this.parent);  // propagate up if dimensions changed
    }

    if (this._containsFlexGrow === true) {
      // Second pass for flexGrow children
      this.children.forEach((c) => {
        if (c.display === 'flex' && isElementNode(c)) {
          calculateFlex(c);
          isFunc(c.onLayout) && c.onLayout.call(c, c);
          addToLayoutQueue(this);
        }
      });
    }
  }
}
```

Key behaviors:

1. **Skip guard:** If `display === 'flex'` and `flexGrow` is set and `width === 0`, returns immediately — the container's own size isn't resolved yet
2. **Flex calculation:** `calculateFlex(this)` returns `true` if any dimensions changed
3. **Self-removal:** Removes self from `layoutQueue` after processing to prevent re-processing in the same run
4. **Upward propagation:** If flex or `onLayout` changed something, adds parent to layout queue so changes bubble up
5. **flexGrow second pass:** When `_containsFlexGrow === true`, recalculates each flex child after the parent layout — prevents infinite loops by calling `calculateFlex` directly rather than adding children to the queue

`requiresLayout()` returns `true` if `display === 'flex'` OR `onLayout` is set — both trigger layout queue inclusion.

The `calculateFlex` function is selected at module load time based on `VITE_USE_NEW_FLEX` env var — see [[VITE_USE_NEW_FLEX selects between old and new flex layout engines]].

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[performance guide]]
