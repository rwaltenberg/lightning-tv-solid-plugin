---
description: The style setter skips any key that is already defined on the element, giving JSX inline props higher specificity than the style object; Config.lockStyles prevents re-application
type: gotcha
module: core
created: 2026-04-07
---

# style setter only applies properties not already set by JSX props

When a `style` object is assigned to an `ElementNode`, it does NOT blindly overwrite all properties. The setter iterates the style object and skips any key where the element already has a value (including `0`):

```typescript
set style(style: Styles | undefined) {
  // dev warning if set twice
  // Config.lockStyles prevents re-set
  this._style = style;
  for (const key in this._style) {
    // be careful of 0 values
    if (this[key as keyof Styles] === undefined) {
      this[key as keyof Styles] = this._style[key as keyof Styles];
    }
  }
}
```

This means JSX inline props take priority over the `style` object:

```jsx
// w from style (100) is ignored; JSX w (200) wins
<View style={{ w: 100, color: 0xff0000ff }} w={200} />
// result: w = 200, color = 0xff0000ff
```

**`Config.lockStyles`:** When enabled, the style setter is a no-op after the first assignment. Useful in performance-critical scenarios to prevent style churn.

**Dev mode warning:** In dev mode, setting `style` twice on the same node logs a console warning with a link to the docs. This catches accidental double-style-setting patterns.

**`_undoStyles`:** The `_stateChanged()` method uses `_style` as the rollback source — when states clear, it reads from `this.style[key]` to restore previous values. If a key is missing from `_style`, a dev warning fires.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
