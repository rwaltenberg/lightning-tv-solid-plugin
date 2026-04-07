# selectedNode Getter Mutates selected

> Reading the selectedNode getter has a side effect: it scans forward from the current selected index and writes this.selected to the first ElementNode it finds. Accessing it in reactive contexts can silently change selection state.

**Source**: `elementNode.ts` (lines 831-842) | **Severity**: important

## Detail

The `selectedNode` getter is not a pure read. It iterates from the current `selected` index, skipping non-ElementNode children (like TextNode), and **writes `this.selected = i`** when it finds the first ElementNode.

```ts
get selectedNode(): ElementNode | undefined {
  const selectedIndex = this.selected || 0;
  for (let i = selectedIndex; i < this.children.length; i++) {
    const element = this.children[i];
    if (isElementNode(element)) {
      this.selected = i;   // SIDE EFFECT: mutates state
      return element;
    }
  }
  return undefined;
}
```

### Why This Matters

- **Reactive contexts**: Accessing `selectedNode` inside `createEffect` or `createMemo` will silently increment `selected` past any TextNode children, which may trigger downstream effects.
- **Debugging**: Logging `container.selectedNode` during development changes the selection state you're trying to inspect.
- **Non-ElementNode children**: If your container mixes `<text>` and `<view>` children, the selected index will skip over text nodes permanently -- there's no way to select a TextNode.

## Code Example

```tsx
// Surprising: reading selectedNode changes selected
const container = <view forwardFocus>
  <text>Label</text>    {/* index 0: TextNode, skipped */}
  <view width={100} height={50} />  {/* index 1: ElementNode */}
  <view width={100} height={50} />  {/* index 2: ElementNode */}
</view>;

// container.selected starts at 0
console.log(container.selected);      // 0
console.log(container.selectedNode);  // ElementNode at index 1
console.log(container.selected);      // 1 (CHANGED by the getter!)
```

## Gotchas

- `selectedNode` is used internally by Row, Column, Grid, and navigation primitives. These components work correctly because they expect this behavior. The gotcha is for custom code that reads `selectedNode` without expecting the mutation.
- `selected || 0` means a `selected` value of `0` is treated the same as `undefined` (both start scanning from index 0).

## Related Notes

- [api/row.md] -- Row uses selectedNode internally
- [api/column.md] -- Column uses selectedNode internally
- [focus/forward-focus-required.md] -- containers must declare forwardFocus
