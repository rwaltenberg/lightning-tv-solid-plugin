---
description: render() is called when a node's parent becomes rendered, converting accumulated props into a live Lightning renderer node and recursively rendering children
type: api
module: core
created: 2026-04-07
---

# ElementNode render method controls the two-phase lifecycle from accumulation to live rendering

`render(topNode?: boolean)` is the method that converts an `ElementNode` from prop-accumulation mode into a live rendered node. It is called when an element is inserted into a tree that is already rendered.

## Execution order

1. **Guard checks** — returns early if parent is not set or not rendered; warns in console
2. **Layout queuing** — if parent requires layout, adds it to the layout queue
3. **Re-render path** — if `this.rendered` is already true (node is being reordered in an array), fires `onRender` and returns early; layout was already queued
4. **State application** — if `_states` exists, calls `_stateChanged()` to apply state styles
5. **Position computation** — handles `right`, `bottom`, `center`, `centerX`, `centerY` to derive `x`, `y`, `mountX`, `mountY` from parent dimensions
6. **Node creation** — for text nodes: applies font defaults, resolves `contain` into `maxWidth`/`maxHeight`, creates via `renderer.createTextNode()`. For element nodes: defaults NaN `w`/`h` to parent dimensions, defaults `color` to transparent, creates via `renderer.createNode()`
7. **Sets `rendered = true`**
8. **Lifecycle hooks** — calls `onCreate` then `onRender`
9. **Event binding** — binds `onEvent` handlers to `lng.on()`
10. **Recursive children render** — iterates children and calls `c.render()` for each `ElementNode`
11. **Initial layout pass** — if `topNode` is true, schedules one `runLayout` via `queueMicrotask`

## Key behaviors

- Elements without a `texture` get default NaN-handling: `w = flexGrow ? 0 : parentWidth - x`; `h = parentHeight - y`
- Elements with `rtt` (render-to-texture) get `color = 0xffffffff` automatically if no color is set
- Elements without `color` or `src` default to transparent `0x00000000`
- Text nodes with `textAlign` but no `contain` get a dev warning

The `topNode` flag tells render to schedule the initial layout pass — this is passed as `true` when the root node is first rendered.

Since [[ElementNode lng property holds a plain object before render and the live renderer node after]], all props set before this method runs are forwarded to the renderer in the `props` object passed to `createNode`.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
