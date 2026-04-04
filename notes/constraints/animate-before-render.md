# Animate Before Render

> animate() asserts this.rendered in dev mode. Use onCreate or onRender callbacks for initial animations. onDestroy supports async animation patterns.

**Source**: `solidjs-integration.md`, `core-rendering-nodes.md` | **Severity**: critical

## Detail

The `animate()` method includes an assertion: `assertTruthy(this.rendered, 'Node must be rendered before animating')`.

In dev mode, calling `animate()` before the node has been rendered throws an error. In production, it is undefined behavior because `this.lng` does not yet exist as an `INode` (it is still a plain object accumulating props).

### Why animate() Requires rendered === true

`this.lng` is a plain object before `render()`. After `render()`, `this.lng` becomes a live `INode` / `IRendererNode` with an `animateProp` method. Animations require this live node to exist.

### Correct Patterns

**For entrance animations**: Use the `onCreate` callback, which fires after `render()` completes (step 8 in the render sequence).

**For exit animations**: Use the `onDestroy` callback, which supports returning a `Promise`. Destruction is deferred until the Promise resolves.

### onDestroy Async Pattern

```ts
onDestroy?: (this: ElementNode, el: ElementNode) => Promise<any> | void;
```

If `onDestroy` returns a `Promise`, `destroy()` waits for the Promise to resolve before calling `_destroy()` (which frees GPU resources).

## Code Example

```tsx
// WRONG -- assertTruthy fails in dev, undefined behavior in prod
const node = new ElementNode('node');
node.animate({ x: 100 }, { duration: 300 }); // NOT RENDERED YET

// CORRECT -- entrance animation via onCreate
<view
  onCreate={(el) => {
    el.animate({ alpha: 1 }, { duration: 300 }).start();
  }}
  alpha={0}
/>

// CORRECT -- exit animation via onDestroy (defers GPU cleanup)
<view
  onDestroy={(el) => {
    return el.animate({ alpha: 0 }, { duration: 300 }).start().waitUntilStopped();
  }}
/>

// CORRECT -- onRender also safe (fires if already rendered on re-insert)
<view
  onRender={(el) => {
    el.animate({ scale: 1.0 }, { duration: 200 }).start();
  }}
/>
```

## Gotchas

- `onCreate` fires at step 8 in `render()`, before children are recursively rendered (step 11). If you need children to be rendered first, use `onLayout` or a microtask.
- `onRender` fires at step 9 in `render()`, and ALSO fires when an already-rendered node is re-inserted into a rendered parent (the early-return path). Be careful about triggering animations twice.
- `onDestroy` returning a non-Promise value destroys the node immediately.

## Related Notes

- [core/node-lifecycle.md] -- full render() sequence (onCreate at step 8, onRender at step 9)
- [core/element-node-proxy.md] -- why this.lng must be a live INode for animations
- [constraints/chain-while-running.md] -- animation chain behavior
