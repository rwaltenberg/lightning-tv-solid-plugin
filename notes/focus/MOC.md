# Focus & Navigation -- D-pad Navigation System

The focus system is a custom tree-walk built entirely in userland. No browser focus model, no tabindex, no :focus pseudo-class.

## Notes

- [forward-focus-required](forward-focus-required.md) -- containers MUST declare forwardFocus (most common mistake)
- [use-focus-manager-once](use-focus-manager-once.md) -- call useFocusManager() exactly once at app root
- [key-handler-return-true](key-handler-return-true.md) -- must return true to stop propagation, false/void continues
- [two-phase-propagation](two-phase-propagation.md) -- capture (root->leaf) then bubble (leaf->root)
- [default-key-map](default-key-map.md) -- ArrowLeft->Left, Enter->Enter, Backspace->Back, etc.
- [handler-signature-differences](handler-signature-differences.md) -- onLeft(e,target,elm) vs onKeyPress(e,mapped,elm,focused)
- [set-focus-microtask](set-focus-microtask.md) -- queueMicrotask coalescing; only last call wins
- [set-focus-unrendered](set-focus-unrendered.md) -- deferred via _autofocus; prefer autofocus prop
- [skip-focus](skip-focus.md) -- decorative elements need skipFocus={true}
- [set-active-element-forbidden](set-active-element-forbidden.md) -- never call setActiveElement directly
- [focus-path](focus-path.md) -- ordered array leaf-to-root; $focus state added/removed automatically
- [spatial-navigation](spatial-navigation.md) -- spatialForwardFocus + spatialHandleNavigation for flex-wrap grids
- [plinko-navigation](plinko-navigation.md) -- Column preserves Row's selected index on vertical nav
- [focus-stack](focus-stack.md) -- FocusStackProvider/useFocusStack for modal overlay focus save/restore
- [input-throttling](input-throttling.md) -- global vs per-element; keyup never globally throttled
