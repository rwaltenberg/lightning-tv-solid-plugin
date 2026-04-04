# Unsized Text Breaks Flex

> An ElementText with .text set but no width/height causes flex calculation to abort (return false). Dimensions must be resolved before flex layout runs.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

When a `<text>` node (`_type: 'textNode'`) has its `text` property set but has not yet resolved its width/height (i.e., the text renderer has not yet reported dimensions), the flex layout calculation aborts and returns `false`.

This happens because the flex engine needs concrete dimensions for all children to calculate layout. An unsized text node -- one where the text is set but dimensions are still `NaN` or `undefined` -- represents an unresolved dependency that prevents layout from completing.

### The Failure Mode

The flex calculation returns `false` (aborting) when it encounters a text child whose dimensions are not yet available. This means the parent flex container's layout is not applied until the text node resolves its dimensions.

### Resolution Pattern

Dimensions must be resolved via the `loaded` callback (from `onEvent`) before flex layout can proceed. The `loaded` event fires when the Lightning text renderer has finished measuring the text and reports `width` and `height`.

## Code Example

```tsx
// WRONG -- text node has no dimensions at render time, breaks flex parent
<view display="flex" flexDirection="row">
  <text>Dynamic text</text>
  <view w={100} h={50} />
</view>

// CORRECT -- set explicit w/h on text node
<view display="flex" flexDirection="row">
  <text w={200} h={30}>Dynamic text</text>
  <view w={100} h={50} />
</view>

// CORRECT -- use contain to set bounding box for text
<view display="flex" flexDirection="row">
  <text contain="width" w={200}>Dynamic text</text>
  <view w={100} h={50} />
</view>

// CORRECT -- trigger layout update after text loads
<view display="flex" flexDirection="row">
  <text
    onEvent={{
      loaded: (el) => el.parent?.updateLayout()
    }}
  >Dynamic text</text>
  <view w={100} h={50} />
</view>
```

## Gotchas

- `autosize` on a text node lets the renderer set width/height after measuring -- but layout still needs to wait for the `loaded` event.
- Text nodes inside a flex container should either have explicit `w`/`h`, use `contain`, or trigger `updateLayout()` on the parent after the `loaded` event.
- The flex engine abort (`return false`) is silent in production.

## Related Notes

- [core/width-height-defaults.md] -- NaN w/h behavior for non-text nodes
- [core/node-lifecycle.md] -- render() sequence for textNode type
- [constraints/textalign-requires-contain.md] -- contain is also required for textAlign
