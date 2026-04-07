---
description: When forwardStates is true, every _stateChanged() call copies the parent's states array to all children before applying the parent's own style changes
type: api
module: core
created: 2026-04-07
---

# forwardStates propagates parent state array to all children on every state change

When `forwardStates = true` is set on an `ElementNode`, every state change on the parent is propagated to all children:

```typescript
_stateChanged() {
  if (this.forwardStates) {
    const states = this.states.slice() as States;
    this.children.forEach((c) => {
      c.states = states;
    });
  }
  // ... then apply styles to self
}
```

The states are sliced (copied) before propagation so children get an independent copy.

**Use case:** Focus rings, highlighting, or group visual states where multiple children need to react to the parent's state. For example, a list item container with `forwardStates` ensures both the background and text child apply `$focus` styling when the item receives focus.

```jsx
// Parent row item - gets $focus when selected
<View forwardStates style={containerStyle}>
  <Text style={textStyle} />  {/* also gets $focus applied */}
  <Image style={imageStyle} />  {/* also gets $focus applied */}
</View>
```

**Propagation depth:** Only propagates one level — to direct children. Children do not automatically propagate further. To go deeper, set `forwardStates` on child containers as well.

**State merge:** Since the `states` setter merges (via `States.merge()`), children keep any states they had and gain the parent's states. This can lead to accumulation if not managed carefully.

---

Related Entries:
- [[dollar-prefix state keys in NodeStyles apply style variants based on active states]] — forwardStates ensures children can react to `$focus` and other states without receiving focus themselves
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — list items inside Row commonly use forwardStates so the item's background, text, and image children all respond to `$focus` styling together
- [[borderBox directive injects a focus ring child into Lightning nodes using onFocusChanged]] — an alternative approach to focus feedback that uses a dedicated child node rather than state propagation

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
- [[focus guide]]
