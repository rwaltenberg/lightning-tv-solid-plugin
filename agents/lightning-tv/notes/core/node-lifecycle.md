# Node Lifecycle

> Full lifecycle of an ElementNode: creation, property accumulation, tree insertion, render(), updates, destruction.

**Source**: `core-rendering-nodes.md` | **Severity**: critical

## Detail

### Phase 1: Creation

```
new ElementNode(name)
  |
  _type = 'element' | 'textNode'
  rendered = false
  lng = {}          <-- plain object, accumulates props
  children = []
```

`name === 'text'` produces `_type = 'textNode'`; anything else produces `_type = 'element'`.

### Phase 2: Property Accumulation (Pre-Render)

SolidJS sets properties on the node. Because `this.lng` is a plain object at this point, property setters write directly to it. No animations or transitions fire -- the `rendered` guard in `_sendToLightningAnimatable` prevents it.

Style is applied once: keys from the style object are copied to the element only if the element does not already have a value for that key. JSX props take priority.

### Phase 3: Tree Insertion

```
parent.insertChild(node, beforeNode?)
  |
  node.parent = parent
  if (beforeNode) --> splice into correct position
  else --> push to children array
```

If the node already had a parent, `removeChild` is called on the old parent first. If the new parent is not yet rendered but the node was previously rendered, `_hasRenderedChildren` is set on the parent.

### Phase 4: Rendering (`render()`)

Preconditions: `parent` must exist and `parent.rendered` must be `true`.

```
render(topNode?)
  |
  1. Queue parent for layout if needed
  2. If already rendered --> call onRender, return
  3. Apply states (_stateChanged)
  4. Resolve positioning (right, bottom, center, centerX, centerY)
  5. Set props.parent = parent.lng
  |
  ├── If textNode:
  |     Apply Config.fontSettings defaults
  |     Resolve text content (getText())
  |     Handle contain/maxWidth/maxHeight
  |     Convert shader if needed
  |     node.lng = renderer.createTextNode(props)
  |
  └── If element:
        Default w/h to parent dimensions minus offset (if NaN)
        Default color to transparent (0x00000000) if no src/texture
        Convert shader if needed
        node.lng = renderer.createNode(props)
        Re-parent any previously rendered children
  |
  6. node.rendered = true
  7. Store _rendererProps (DEV only)
  8. Call onCreate(this)
  9. Call onRender(this)
  10. Attach onEvent handlers to lng node
  11. Recursively render children
  12. If topNode, queue layout via microtask
  13. If autofocus, call setFocus()
```

### Phase 5: Updates (Post-Render)

- **Animatable props**: go through `_sendToLightningAnimatable`, which checks for `transition` config. If a transition exists, calls `this.lng.animateProp(name, value, settings)`. Otherwise, sets `this.lng[name] = value` directly.
- **Non-animating props**: set directly via `this.lng[key] = v`.
- **Shader/effect props**: go through `shaderAccessor`, which updates `_effects`, recomputes shader props, and creates or updates the shader.

### Phase 6: Destruction

```
destroy()
  |
  1. If onDestroy is defined, call it
  2. If onDestroy returns a Promise, wait for it to resolve
  3. Call _destroy()
       |
       If this.lng is an INode (has destroy method), call this.lng.destroy()
```

### Lifecycle Callbacks

```ts
onCreate?: (this: ElementNode, el: ElementNode) => void;
onDestroy?: (this: ElementNode, el: ElementNode) => Promise<any> | void;
onRender?: (this: ElementNode, el: ElementNode) => void;
onRemove?: (this: ElementNode, el: ElementNode) => void;
onLayout?: (this: ElementNode, target: ElementNode) => void;
onEvent?: OnEvent;  // { loaded?, failed?, freed?, inBounds?, outOfBounds?, inViewport?, outOfViewport? }
```

`onDestroy` supports async cleanup -- if it returns a `Promise`, destruction is deferred until the Promise resolves.

### Focus Lifecycle

```
setFocus()
  |
  If rendered:
    If forwardFocus is a function --> call it; if returns !== false, stop
    If forwardFocus is a number --> setFocus on that child index
    Otherwise --> queue nextActiveElement via microtask
  If not rendered:
    Set _autofocus = true (will fire on render)
```

## Code Example

```tsx
<view
  onCreate={(el) => {
    // Node is rendered -- safe to animate
    el.animate({ alpha: 1 }, { duration: 300 }).start();
  }}
  onRender={(el) => {
    // Also called when already-rendered node re-renders
    console.log('rendered', el.id);
  }}
  onDestroy={(el) => {
    // Async: destruction waits for this Promise
    return el.animate({ alpha: 0 }, { duration: 300 }).start().waitUntilStopped();
  }}
  onRemove={(el) => {
    // Called when node is removed from parent
  }}
  onLayout={(el) => {
    // Called when flex layout recalculates
  }}
/>
```

## Gotchas

- `onCreate` fires before children are recursively rendered (step 8 before step 11).
- `onRender` also fires when a node that is already rendered is re-inserted into a rendered parent (step 2 early-return path calls `onRender` then returns).
- `onDestroy` returning a Promise defers GPU resource cleanup until the Promise resolves -- useful for exit animations.
- Focus deferred via microtask: multiple `setFocus()` calls in the same microtask only resolve the last one.
- `autofocus` is processed at step 13, after children are rendered.

## Related Notes

- [core/element-node-proxy.md] -- how this.lng transitions from plain object to INode
- [constraints/animate-before-render.md] -- why animate() requires rendered === true
- [core/node-deletion-queue.md] -- how deletion is deferred across microtasks
- [core/width-height-defaults.md] -- NaN w/h behavior during render()
