---
description: View wraps createElement('node') and Text wraps createElement('text'); both use spread() with false to pass props without merging defaults; exported for use in non-JSX contexts
type: gotcha
module: core
created: 2026-04-07
---

# View and Text components avoid JSX to prevent circular dependency issues

`View` and `Text` are Lightning's primitive element components, implemented without JSX in `render.ts`:

```ts
export const View = (props: NodeProps) => {
  const el = createElement('node');
  spread(el, props, false);
  return el as unknown as JSXElement;
};

export const Text = (props: TextProps) => {
  const el = createElement('text');
  spread(el, props, false);
  return el as unknown as JSXElement;
};
```

The comment in the source: "Don't use JSX as it creates circular dependencies and causes trouble with the playground."

The third argument `false` to `spread(el, props, false)` is the `skipUndefined` parameter — passing `false` here means props are spread as-is without skipping undefined values.

`View` creates a `'node'` type element (mapped to `ElementNode` with visual rendering). `Text` creates a `'text'` type element (an `ElementText`, which accepts font and text styling props).

The `Dynamic` component in the same file handles the case where the element type is determined at runtime. For function components, it uses `untrack()` to run the component once without creating a reactive dependency on the memo's internal execution:

```ts
case 'function':
  return untrack(() => component(others));
```

For string types (like `'node'` or custom string names), it creates the element with `createElement` and spreads props.

---

Source: [[render]]
Domains:
- [[core guide]]
