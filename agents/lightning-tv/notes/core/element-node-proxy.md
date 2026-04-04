# Element Node Proxy Pattern

> ElementNode uses Object.defineProperty on the prototype so all rendering properties proxy through to this.lng, which is the source of truth.

**Source**: `core-rendering-nodes.md` | **Severity**: critical

## Detail

`ElementNode` does not store rendering properties internally. Properties like `x`, `y`, `w`, `h`, `color`, `alpha`, etc. are defined via `Object.defineProperty` on the prototype. Their getters read from `this.lng[key]`, and their setters either:

- Call `_sendToLightningAnimatable(key, value)` for animatable numeric props (which may trigger transition animations), or
- Write directly to `this.lng[key]` for non-animating props.

`this.lng` is the **source of truth** for all renderer-visible state. Before `render()`, `this.lng` is a plain object accumulating props. After `render()`, it becomes a live `INode` / `IRendererNode` from the underlying renderer.

### Animatable Properties (go through `_sendToLightningAnimatable`)

`x`, `y`, `w`, `h`, `color`, `alpha`, `scale`, `scaleX`, `scaleY`, `rotation`, `mount`, `mountX`, `mountY`, `pivot`, `pivotX`, `pivotY`, `zIndex`, `fontSize`, `lineHeight`, `colorTop`, `colorRight`, `colorLeft`, `colorBottom`, `colorTl`, `colorTr`, `colorBl`, `colorBr`

### Non-Animating Properties (write directly to `this.lng[key]`)

`autosize`, `clipping`, `contain`, `data`, `text`, `textAlign`, `texture`, `textureOptions`, `fontStretch`, `fontStyle`, `imageType`, `letterSpacing`, `maxHeight`, `maxLines`, `maxWidth`, `offsetY`, `overflowSuffix`, `rtt`, `scrollable`, `scrollY`, `src`, `srcHeight`, `srcWidth`, `srcX`, `srcY`, `strictBounds`, `textBaseline`, `textOverflow`, `verticalAlign`, `wordBreak`, `wordWrap`, `group`, `destroyed`, `preventCleanup`

### Shader/Effect Accessors (via `shaderAccessor`)

`border`, `borderBottom`, `borderTop`, `borderLeft`, `borderRight`, `shadow`, `rounded`, `borderRadius`, `linearGradient`, `radialGradient`, `effects`

## Code Example

```tsx
// Reading a property reads from this.lng (the renderer node)
const node = new ElementNode('node');
node.x = 100;      // calls setter --> this.lng.x = 100 (pre-render: plain object)
console.log(node.x); // getter --> returns this.lng.x

// After render(), this.lng is a live INode
// Animatable props may trigger transition if transition is configured
node.transition = { x: { duration: 300 } };
node.x = 200; // calls _sendToLightningAnimatable --> lng.animateProp('x', 200, settings)
```

## Gotchas

- Before `render()`, `this.lng` is a plain JS object -- no animations fire even for animatable properties (the `rendered` guard in `_sendToLightningAnimatable` prevents it).
- After `render()`, `this.lng` is a live renderer node -- writes to animatable properties go through `animateProp` if `transition` is configured.
- Animatable properties defined on the class interface are NOT defined in the class body -- they live on the prototype via `Object.defineProperty`.

## Related Notes

- [core/rendering-pipeline.md] -- how ElementNode fits into the 4-layer pipeline
- [core/node-lifecycle.md] -- when this.lng transitions from plain object to INode
- [reactivity/signal-to-gpu-pipeline.md] -- full signal -> setter -> lng flow
- [reactivity/reactive-transitions.md] -- transition config that drives animateProp calls
