---
description: NodeType enum defines three values — element, textNode, text — used by isTextNode() and isElementText() utilities to filter nodes during layout and rendering
type: architecture
module: core
created: 2026-04-07
---

# node type discriminators allow the flex engine to skip text nodes

The framework has three distinct node types, identified by a string constant stored on each node's `_type` property. These discriminators drive runtime type narrowing in layout and rendering code.

## NodeType enum

```typescript
export const NodeType = {
  Element: 'element',   // ElementNode — standard rendering node
  TextNode: 'textNode', // ElementText — text display container
  Text: 'text',         // TextNode — raw text string wrapper
} as const;

export type NodeTypes = 'element' | 'textNode' | 'text';
```

## Three node categories

**`'element'`** — The standard `ElementNode`. Has children, supports flex layout, has a renderer-backed rendering target.

**`'textNode'`** — `ElementText` nodes. A text-rendering container that manages display of text content. Has `children: TextNode[]` and a `text` string. NOT a flex item.

**`'text'`** — Bare `TextNode`. The raw string wrapper that contains the actual text value. Children of `ElementText`. Has `[key: string]: any` to accommodate JSX children transformation.

## Usage in layout

```typescript
// In flex.ts and flexLayout.ts:
if (isTextNode(c) || c.flexItem === false) {
  continue; // excluded from flex item processing
}

if (isElementText(c) && c.text && !(c.width || c.height)) {
  return false; // early exit — text size not yet known
}
```

The `isTextNode()` and `isElementText()` utilities from `utils.ts` check `_type` against these constants.

---

Related Entries:
- [[flex engine returns false early when ElementText child has text but no explicit dimensions]] — isElementText check drives this gotcha
- [[flexItem false excludes a child from flex layout calculations]] — text nodes are excluded by the same `continue` block
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — NodeProps is built on CleanElementNode which excludes _type from public interface

Source: [[core-nodeTypes]]
Domains:
- [[core guide]]
