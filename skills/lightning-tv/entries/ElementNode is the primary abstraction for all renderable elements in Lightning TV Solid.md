---
description: ElementNode is a DOM-like class that wraps the Lightning WebGL renderer node, bridging SolidJS reactivity with LightningJS 3 rendering
type: architecture
module: core
created: 2026-04-07
---

# ElementNode is the primary abstraction for all renderable elements in Lightning TV Solid

Every visible element on screen — including text nodes — is an instance of `ElementNode`. It extends `Object` (not HTMLElement) and provides a DOM-like API that maps to the LightningJS 3 WebGL renderer.

The class serves several roles simultaneously:
- A **property accumulator** before render (props are stored on `lng` as a plain object)
- A **live renderer proxy** after render (`lng` becomes the actual renderer node)
- A **layout container** for flex children
- A **focus target** for the focus manager — since [[FocusNode interface defines the focus lifecycle callbacks]], any ElementNode can declare `onFocus`, `onBlur`, and `onFocusChanged` handlers
- A **state machine** for visual state variants — since [[focus state key is automatically added and removed from elm states during focus path changes]], focus-driven style variants activate automatically

```typescript
export class ElementNode extends Object {
  constructor(name: string) {
    super();
    this._type = name === 'text' ? NodeType.TextNode : NodeType.Element;
    this.rendered = false;
    this.lng = {}; // accumulates props until render()
    this.children = [];
  }
}
```

The constructor accepts a name string — `'text'` creates a `TextNode` type, anything else creates an `Element` type. This means both text and element nodes share the same class.

Since [[ElementNode lng property holds a plain object before render and the live renderer node after]], there is a clear two-phase lifecycle: accumulation and rendering. Since [[setFocus defers focus assignment via microtask to allow children to render first]], Row and Column navigation relies on this two-phase lifecycle — children must be rendered before focus can be forwarded to them.

---

Related Entries:
- [[NodeProps and NodeStyles are the primary JSX prop types for element nodes]] — the TypeScript interface that exposes all ElementNode capabilities in JSX
- [[ElementNode render method controls the two-phase lifecycle from accumulation to live rendering]] — the method that transitions the node from accumulation to live state

Source: [[core-elementNode]]
Domains:
- [[core guide]]
