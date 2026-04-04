# Core -- Node System & Rendering Architecture

Notes on ElementNode, the rendering pipeline, lifecycle, and the deletion queue.

## Notes

- [element-node-proxy](element-node-proxy.md) -- property proxy pattern; this.lng is the source of truth
- [rendering-pipeline](rendering-pipeline.md) -- JSX -> solid-js/universal -> ElementNode -> Lightning Renderer
- [node-lifecycle](node-lifecycle.md) -- onCreate, onRender, onLayout, onRemove, onDestroy execution order
- [node-deletion-queue](node-deletion-queue.md) -- microtask-based counter system for safe node reordering
- [width-height-defaults](width-height-defaults.md) -- NaN defaults to parentWidth - x at render time
- [dual-renderer-architecture](dual-renderer-architecture.md) -- WebGL RendererMain vs DOM DOMRendererMain fallback
