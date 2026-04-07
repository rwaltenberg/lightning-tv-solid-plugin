---
description: The states setter merges into the existing States object if one exists, or creates a new one; immediately calls _stateChanged() if already rendered to apply style changes
type: api
module: core
created: 2026-04-07
---

# states setter merges new states into existing States object and triggers immediate style application

The `states` property on `ElementNode` is backed by a `States` object that notifies the node when states change:

```typescript
set states(states: NodeStates) {
  this._states = this._states
    ? this._states.merge(states)
    : new States(this._stateChanged.bind(this), states);
  if (this.rendered) {
    this._stateChanged();
  }
}

get states(): States {
  this._states = this._states || new States(this._stateChanged.bind(this));
  return this._states;
}
```

The getter lazily creates the `States` object on first access. The setter merges if one already exists — meaning assigning states does not reset previously added states.

**Common usage patterns:**

```typescript
// Add a state (States API)
node.states.add('$focus')
node.states.toggle('$disabled')
node.states.remove('$focus')

// Replace/merge
node.states = ['$focus', '$selected']
```

The `_stateChanged.bind(this)` callback is passed to `States` so that any mutation on the `States` object (not just via the setter) triggers the style application logic.

Since [[_stateChanged applies dollar-prefix style variants with undo tracking and state order priority]], this is where the visual transition between states actually happens.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
