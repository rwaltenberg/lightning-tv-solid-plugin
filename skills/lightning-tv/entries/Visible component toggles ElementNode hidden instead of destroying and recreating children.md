---
description: Unlike `<Show>`, `<Visible>` mounts children once and toggles `childNode.hidden` on each condition change, trading memory for faster show/hide cycles
type: comparison
module: primitives
created: 2026-04-07
---

`Visible` is a conditional rendering primitive optimized for components that toggle frequently or are expensive to mount:

```tsx
<Visible when={isActive()}>
  <ExpensivePanel />
</Visible>
```

## How It Differs from Show

| Behavior | `<Show>` | `<Visible>` |
|----------|----------|-------------|
| On first truthy | Mount children | Mount children |
| On falsy | Destroy children | Set `hidden = true` |
| On re-truthy | Remount children | Set `hidden = false` |
| Memory | Releases on hide | Retains always |
| Remount cost | Paid each time | Paid once |

## Implementation

Children are created in a `createRoot` on first truthy condition:
```ts
if (c && !child) {
  disposer = createRoot((dispose) => {
    child = children(() => props.children);
    return dispose;
  });
}
```

On subsequent condition changes, only `hidden` toggles:
```ts
child?.toArray().forEach((childNode) => {
  if (childNode instanceof ElementNode) {
    childNode.hidden = isHidden;
  }
});
```

## keyed Mode

With `keyed: true`, the component behaves more like `<Show keyed>`: the reactive root is disposed and children are re-created whenever `when` changes identity (not just truthiness). Use this when `when` carries data that children depend on and should remount when data changes.

## When to Use

Use `<Visible>` when:
- Children are expensive to mount (complex layouts, images, video)
- The component toggles frequently (e.g., panel sliding in/out)
- Remount would cause flicker or loading states

Use `<Show>` when:
- Hidden state should release memory
- Children are cheap to recreate
- You want cleanup side effects to run on hide

Since [[Preserve component marks an ElementNode as preserve=true to survive route transitions]], `Visible` is the within-page equivalent — both hide rather than destroy, but at different scopes.

---

Source: [[primitives-Visible]]
Domains:
- [[performance guide]]
- [[components guide]]
