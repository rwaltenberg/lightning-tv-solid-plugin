# TextNode Children Are Skipped; Unsized ElementText Aborts Layout

> `TextNode` instances (`_type='text'`) are excluded from flex participation. An `ElementText` child with text but no dimensions causes flex to abort entirely.

**Source**: `flex-layout-engine.md` | **Severity**: critical

## Detail

During Phase 1 (child filtering), the engine applies two text-related rules:

**Rule 1 -- TextNode skip**: If a child is a `TextNode` (a pure text node, not an element) or has `flexItem === false`, it is skipped and not added to the processable list. It does not contribute to `numProcessedChildren`, does not occupy space, and is not positioned.

**Rule 2 -- ElementText bail-out**: If a child is an `ElementText` (an element with `_type === 'textNode'`) with a `.text` value set but neither `width` nor `height` defined, the **entire flex calculation aborts** and returns `false`. This is a guard for text nodes that haven't been measured yet (typically resolved via a `loaded` callback).

These are two different behaviors:
- `TextNode` (pure text) -> silently skipped, layout continues
- `ElementText` with text and no dimensions -> entire layout aborts, returns `false`

## Gotchas

- A flex container with an `ElementText` child that hasn't loaded its dimensions yet will produce no layout output at all -- all children, including non-text children, will remain unpositioned from that layout pass.
- The `loaded` callback pattern is the intended mechanism for triggering re-layout once text dimensions are resolved.
- `TextNode` children being skipped means they do not count toward `numProcessedChildren`. A container that appears to have 2 children (one text, one element) is treated as having 1 processable child, which disables `flexGrow`.

## Related Notes

- [flex/flexgrow-two-children.md] -- `numProcessedChildren > 1` requirement; text nodes reduce effective count
- [flex/explicit-dimensions-required.md] -- children must have explicit dimensions; text nodes are the key case where this is violated
