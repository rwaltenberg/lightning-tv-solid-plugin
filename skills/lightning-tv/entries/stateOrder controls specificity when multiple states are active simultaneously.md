---
description: When multiple states are active, stateOrder (node-level or Config.stateOrder) determines sort order; states later in the array have higher specificity and their styles win conflicts
type: api
module: core
created: 2026-04-07
---

# stateOrder controls specificity when multiple states are active simultaneously

When multiple states are active on a node simultaneously (e.g., both `$focus` and `$selected`), their style objects are merged in a specific order determined by `stateOrder`. The last-applied styles win conflicts.

```typescript
const stateOrder = this.stateOrder || Config.stateOrder;
if (stateOrder && stateOrder.length > 0) {
  sortedStates = states.slice().sort((a, b) => {
    const aIdx = stateOrder.indexOf(a);
    const bIdx = stateOrder.indexOf(b);
    // In stateOrder = higher specificity (applied later)
    if (aIdx !== -1 && bIdx === -1) return 1;   // a later = higher
    if (aIdx === -1 && bIdx !== -1) return -1;  // b later = higher
    return aIdx - bIdx;                          // order by position
  });
}

newStyles = sortedStates.reduce((acc, state) => {
  const styles = this[state];
  return styles ? { ...acc, ...styles } : acc;
}, stylesToUndo || {});
```

States **later** in the `stateOrder` array have higher specificity — their styles are applied last in the reduce and override earlier ones.

**Example:**
```typescript
Config.stateOrder = ['$selected', '$focus', '$disabled']
// $disabled styles win over $focus which wins over $selected
// when multiple are active simultaneously
```

States not in `stateOrder` are treated as lower specificity than any state in the array. Two states both not in `stateOrder` maintain their original array order.

**Node-level override:** `stateOrder` can be set directly on an element to override the global config for that node only.

If no `stateOrder` is configured, states are applied in the order they appear in the `states` array (their insertion order).

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
