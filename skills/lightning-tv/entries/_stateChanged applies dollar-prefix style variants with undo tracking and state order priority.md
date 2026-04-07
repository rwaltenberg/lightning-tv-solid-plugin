---
description: _stateChanged() reads dollar-prefixed keys from the element, merges them in stateOrder priority, applies with Object.assign, and tracks applied keys in _undoStyles for rollback
type: architecture
module: core
created: 2026-04-07
---

# _stateChanged applies dollar-prefix style variants with undo tracking and state order priority

`_stateChanged()` is the internal method that applies visual state changes. It is bound to the `States` object and called on any state mutation.

## Execution flow

1. **forwardStates:** If `this.forwardStates === true`, propagates current states to all children before doing anything else
2. **Skip check:** If neither `_undoStyles` has content nor any current state key exists on the element (`keyExists(this, states)`), returns early — nothing to do
3. **Undo styles:** Builds `stylesToUndo` from `_style` using the previously applied keys in `_undoStyles` — this restores base style values when states are removed
4. **State application:**
   - Zero states: applies `stylesToUndo` (restores base styles), clears `_undoStyles`
   - One state: reads `this['$stateName']` directly
   - Multiple states: sorts by `stateOrder` (node-level or `Config.stateOrder`), reduces into merged styles

```typescript
sortedStates = states.slice().sort((a, b) => {
  const aIdx = stateOrder.indexOf(a);
  const bIdx = stateOrder.indexOf(b);
  if (aIdx !== -1 && bIdx === -1) return 1;  // in stateOrder = higher specificity
  if (aIdx === -1 && bIdx !== -1) return -1;
  return aIdx - bIdx;
});
// Later in stateOrder = applied last = wins on conflict
```

5. **Transition first:** If new styles include a `transition` key, applies it before the other properties (so animated props in the same state change are animated correctly)
6. **Object.assign(this, newStyles)** — applies all style properties to the element, triggering animatable proxies as needed
7. **`_undoStyles`** stores the applied keys for next rollback

**Gotcha:** The `_style` object is the rollback source. If a key was applied via a state but is NOT in `_style`, the dev warning `'fallback style key not found'` fires — and the value cannot be restored properly.

Since [[style setter only applies properties not already set by JSX props]], the base `_style` must be set before states to ensure proper rollback. Since [[states setter merges new states into existing States object and triggers immediate style application]], mutations via `node.states.add()` also trigger this path.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
