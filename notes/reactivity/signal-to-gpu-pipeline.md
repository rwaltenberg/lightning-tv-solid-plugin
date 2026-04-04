# Signal to GPU Pipeline

> Signal change -> SolidJS effect -> setProperty -> node setter -> _sendToLightningAnimatable -> lng.animateProp or lng[key]=value.

**Source**: `solidjs-integration.md` | **Severity**: critical

## Detail

Because SolidJS's compiler wraps reactive expressions in effects, signal changes trigger fine-grained updates. The full path for a numeric/visual property:

```
Signal change (setX(100))
    |
    v
SolidJS effect re-runs
    |
    v
setProperty(node, 'x', 100)   [solidOpts.ts]
    |
    v
node.x = 100                   [ElementNode prototype setter]
    |
    v
_sendToLightningAnimatable('x', 100)
    |
    ├── if transition configured for 'x':
    |     lng.animateProp('x', 100, animationSettings)  --> GPU animation
    |
    └── else:
          lng.x = 100    --> direct GPU write
```

### Text Update Path

For text content:

```
Signal change (setCount(1))
    |
    v
SolidJS effect re-runs for text expression
    |
    v
replaceText(textNode, '1')     [solidOpts.ts]
    |
    v
node.text = '1'                [TextNode object updated]
parent.text = parent.getText() [concatenates all TextNode children]
    |
    v
ElementNode.text setter
    |
    v
this.lng.text = value          [non-animating, direct write to renderer]
```

### Zero-Overhead Reactive Binding

SolidJS's fine-grained reactivity means only the specific property that changed triggers an update. There is no virtual DOM diffing or batch reconciliation overhead. Each signal change results in exactly one setter call on the affected `ElementNode`.

### Alias Resolution in `_sendToLightningAnimatable`

The method resolves aliases before checking `transition`. Width/height props may have aliases (`w` vs `width`). The transition config key must match the resolved name.

## Code Example

```tsx
const [xPos, setXPos] = createSignal(0);
const [count, setCount] = createSignal(0);

// Numeric prop -- goes through _sendToLightningAnimatable
<view
  x={xPos()}
  transition={{ x: { duration: 300, easing: 'ease-out' } }}
/>

// When setXPos(100) is called:
// SolidJS effect -> setProperty(node, 'x', 100) -> node.x = 100
// -> _sendToLightningAnimatable('x', 100)
// -> lng.animateProp('x', 100, { duration: 300, easing: 'ease-out' })

// Text content update
<text>Count: {count()}</text>
// When setCount(5) is called:
// replaceText(textNode, '5') -> parent.text = 'Count: 5' -> lng.text = 'Count: 5'
```

## Gotchas

- Before `node.rendered`, `_sendToLightningAnimatable` writes directly to `lng[key]` even for animatable props -- the rendered guard prevents premature animation.
- `transition` must be configured on the node for `animateProp` to be called; otherwise it's always a direct write.
- Text updates go through `replaceText` -> `parent.text = parent.getText()` which concatenates all TextNode children -- not per-TextNode updates.

## Related Notes

- [core/element-node-proxy.md] -- prototype setter definitions for animatable vs non-animating props
- [reactivity/reactive-transitions.md] -- transition prop configuration
- [core/node-lifecycle.md] -- rendered guard in _sendToLightningAnimatable
