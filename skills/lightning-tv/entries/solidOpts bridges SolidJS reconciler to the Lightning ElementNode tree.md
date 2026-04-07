---
description: The nodeOpts object implements SolidRendererOptions — the 9-method interface that tells SolidJS how to create, insert, remove, and traverse Lightning nodes
type: architecture
module: core
created: 2026-04-07
---

# solidOpts bridges SolidJS reconciler to the Lightning ElementNode tree

`solidOpts.ts` exports the `nodeOpts` object passed to `solidCreateRenderer<SolidNode>(nodeOpts)`. This object is the complete adapter between SolidJS's universal reconciler and the Lightning node tree.

The nine required methods:

| Method | What it does |
|--------|-------------|
| `createElement(name)` | Creates a new `ElementNode(name)` |
| `createTextNode(text)` | Creates a plain `{ _type: NodeType.Text, text }` object |
| `replaceText(node, value)` | Updates text and recalculates parent ElementText content |
| `setProperty(node, name, value)` | Sets `node[name] = value` directly |
| `insertNode(parent, node, anchor)` | Inserts child; triggers render if parent is rendered |
| `isTextNode(node)` | Returns `isElementText(node)` — true for `<text>` nodes |
| `removeNode(parent, node)` | Removes child; defers destruction via delete queue |
| `getParentNode(node)` | Returns `node.parent` |
| `getFirstChild(node)` | Returns `node.children[0]` |
| `getNextSibling(node)` | Finds next sibling by index in parent.children |

Note that `isTextNode` identifies `<text>` JSX nodes (ElementText), not raw text nodes. This is counterintuitive — SolidJS uses it to determine if a node can hold raw text children.

Since [[ElementNode render method controls the two-phase lifecycle from accumulation to live rendering]], `insertNode` triggers `node.render(true)` only when the parent is already rendered (`node.parent!.rendered`). New insertions into unrendered trees accumulate props without rendering.

---

Source: [[solidOpts]]
Domains:
- [[core guide]]
