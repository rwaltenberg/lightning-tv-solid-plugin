---
description: contain='width' sets maxWidth from w; contain='both' sets both maxWidth and maxHeight; textAlign requires contain; missing contain with maxLines=1 uses lineHeight or fontSize as maxHeight
type: api
module: core
created: 2026-04-07
---

# text node contain prop maps to maxWidth and maxHeight during render

During `render()`, text nodes process the `contain` property to derive `maxWidth` and `maxHeight` for the renderer. The full logic:

```typescript
if (textProps.contain) {
  if (textProps.contain === 'both') {
    textProps.maxWidth = textProps.maxWidth ?? textProps.w;
    textProps.maxHeight = textProps.maxHeight ?? textProps.h;
  } else if (textProps.contain === 'width') {
    textProps.maxWidth = textProps.maxWidth ?? textProps.w;
  }

  if (!textProps.h && !textProps.maxHeight) {
    textProps.maxLines = textProps.maxLines ?? 99;
  }

  if (!textProps.maxWidth) {
    textProps.maxWidth = parentWidth - textProps.x! - (textProps.marginRight || 0);
  }

  if (textProps.contain === 'both' && !textProps.maxHeight) {
    textProps.maxHeight = parentHeight - textProps.y! - (textProps.marginBottom || 0);
  } else if (textProps.maxLines === 1) {
    textProps.maxHeight = (textProps.maxHeight || textProps.lineHeight || textProps.fontSize) as number;
  }
}
```

Key behaviors:
- `contain = 'width'`: clamps text to `w` (or derives from parent width - x - marginRight if no explicit maxWidth set)
- `contain = 'both'`: clamps to both dimensions
- No `h` and no `maxHeight`: defaults to 99 max lines
- `maxLines = 1`: `maxHeight` is set to `lineHeight` or `fontSize` as fallback
- `maxWidth` fallback: `parentWidth - x - marginRight` — derived from layout context

**textAlign requires contain (dev warning):**
```typescript
if (textProps.textAlign && !textProps.contain) {
  console.warn('Text align requires contain: ', node.getText());
}
```

The text renderer needs a bounded width to center/right-align text. Without `contain`, text alignment has no reference dimension.

**Post-render layout trigger:** If a text node is in a flex container but `maxWidth` or `maxHeight` isn't determined at render time, `_layoutOnLoad()` is called — the parent's `updateLayout()` is triggered after the text's `loaded` event fires with its rendered dimensions.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
