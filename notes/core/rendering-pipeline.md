# Rendering Pipeline

> Four-layer pipeline: JSX source -> SolidJS compiler -> solid-js/universal -> ElementNode -> Lightning Renderer.

**Source**: `solidjs-integration.md` | **Severity**: critical

## Detail

The pipeline has four layers:

```
JSX Source Code
    |
    v
SolidJS Compiler (transforms JSX into createComponent / createElement / insert calls)
    |
    v
solid-js/universal runtime (calls into nodeOpts: createElement, insertNode, removeNode, etc.)
    |
    v
ElementNode (framework's virtual node layer -- accumulates props)
    |
    v
Lightning Renderer (WebGL/Canvas INode / ITextNode -- actual GPU rendering)
```

### Layer Details

**Layer 1 -- SolidJS Compiler**: Transforms JSX into calls to `createElement`, `createTextNode`, `insertNode`, `setProp`, etc.

**Layer 2 -- solid-js/universal**: `createRenderer` from `solid-js/universal` is called with a configuration object (`nodeOpts` from `src/solidOpts.ts`). This produces a bound renderer with `render`, `effect`, `memo`, `createComponent`, `createElement`, `createTextNode`, `insertNode`, `insert`, `spread`, `setProp`, `mergeProps`, and `use`.

**Layer 3 -- ElementNode**: Every JSX element becomes an `ElementNode` instance. This is NOT a DOM element. `ElementNode` extends `Object` (not `HTMLElement`). It accumulates properties on an internal `lng` object. Instances are created inside-out (children first) by SolidJS's reconciler, but rendered outside-in.

**Layer 4 -- Lightning Renderer**: Rendering is triggered when an `ElementNode` is inserted into a parent that is already rendered (`node.rendered === true`). At render time, `ElementNode.render()` calls either `renderer.createNode(props)` or `renderer.createTextNode(props)`.

### nodeOpts from solidOpts.ts

```typescript
{
  createElement(name: string): ElementNode;
  createTextNode(text: string): TextNode;          // Returns { _type: 'text', text }
  replaceText(node: TextNode, value: string): void; // Updates text and triggers parent re-render
  setProperty(node: ElementNode, name: string, value: any): void; // node[name] = value
  insertNode(parent: ElementNode, node: SolidNode, anchor: SolidNode): void;
  isTextNode(node: SolidNode): boolean;
  removeNode(parent: ElementNode, node: SolidNode): void;
  getParentNode(node: SolidNode): ElementNode | ElementText | undefined;
  getFirstChild(node: ElementNode): SolidNode | undefined;
  getNextSibling(node: SolidNode): SolidNode | undefined;
}
```

### Root Bootstrap Sequence

```typescript
const { renderer, rootNode, render } = createRenderer(
  undefined,  // uses Config.rendererOptions if undefined
  'app'       // DOM element ID or HTMLElement
);
const dispose = render(() => <App />);
```

Internally:
1. `createRenderer` calls `startLightningRenderer(options, 'app')` which instantiates `lng.RendererMain` (or `DOMRendererMain`).
2. The Lightning renderer's root node is assigned to `rootNode.lng`, and `rootNode.rendered = true`.
3. `Config.setActiveElement` is wired to the `setActiveElement` signal setter from `activeElement.ts`.
4. An `'idle'` listener on the renderer enables the task queue.
5. `render(code)` calls `solidRenderer.render(code, rootNode)`.

### JSX Tag to Node Type Mapping

| JSX Tag | ElementNode._type | Lightning Renderer Call |
|---------|------------------|----------------------|
| `<node>` | `'element'` | `renderer.createNode(props)` |
| `<view>` | `'element'` | `renderer.createNode(props)` (alias) |
| `<text>` | `'textNode'` | `renderer.createTextNode(props)` |

## Code Example

```tsx
import { createRenderer, Config } from '@lightningtv/solid';
import { WebGlCoreRenderer } from '@lightningjs/renderer/webgl';
import { CanvasTextRenderer } from '@lightningjs/renderer/canvas';

// 1. Configure BEFORE createRenderer
Config.rendererOptions = {
  fontEngines: [CanvasTextRenderer],
  renderEngine: WebGlCoreRenderer,
};

// 2. Create the renderer
const { renderer, rootNode, render } = createRenderer(
  undefined,
  'app'
);

// 3. Render the app
const dispose = render(() => <App />);
```

## Gotchas

- `ElementNode` is NOT a DOM element -- it extends `Object`, not `HTMLElement`. DOM APIs do not work on it.
- Children are created before parents by SolidJS's reconciler, but rendering cascades outside-in from the root.
- `rootNode.rendered = true` is the seed that allows the entire tree to render.

## Related Notes

- [core/element-node-proxy.md] -- how ElementNode proxies properties to this.lng
- [core/node-lifecycle.md] -- full lifecycle including the render() method
- [core/dual-renderer-architecture.md] -- the two renderer backends
