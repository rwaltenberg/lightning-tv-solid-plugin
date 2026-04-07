# Constraints -- Anti-Patterns & Hard Rules

Notes on things that crash, fail silently, or produce invisible/corrupt output.

## Notes

- [style-locked-once](style-locked-once.md) -- Config.lockStyles prevents re-assignment; never use reactive style prop
- [animate-before-render](animate-before-render.md) -- animate() asserts this.rendered; use onCreate/onRender
- [src-color-side-effect](src-color-side-effect.md) -- setting src auto-sets color only when already rendered
- [textalign-requires-contain](textalign-requires-contain.md) -- textAlign without contain="width" is silently ignored
- [unsized-text-breaks-flex](unsized-text-breaks-flex.md) -- ElementText without dimensions aborts entire flex calc
- [children-array-immutable](children-array-immutable.md) -- must use insertChild/removeChild, never mutate directly
- [chain-while-running](chain-while-running.md) -- calling chain() on running animation clears the queue
- [create-tag-cleanup](create-tag-cleanup.md) -- createTag nodes have preventCleanup: true, must call .destroy()
- [shader-transitions-broken](shader-transitions-broken.md) -- border/shadow/rounded transitions use broken code path; linearGradient/radialGradient have no transition support
- [states-merge-pitfalls](states-merge-pitfalls.md) -- merge() doesn't trigger onChange; array/string form clears all states
- [selected-node-side-effect](selected-node-side-effect.md) -- selectedNode getter mutates selected index as side effect
- [rtt-color-defaults](rtt-color-defaults.md) -- RTT defaults to white; 0x00000000 is falsy and gets auto-overridden
