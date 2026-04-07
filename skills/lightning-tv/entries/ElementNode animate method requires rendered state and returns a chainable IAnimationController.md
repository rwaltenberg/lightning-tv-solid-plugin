---
description: animate(props, animationSettings?) calls the renderer's animate directly and returns IAnimationController; asserts rendered in dev; falls back to node then Config animation settings
type: api
module: core
created: 2026-04-07
---

# ElementNode animate method requires rendered state and returns a chainable IAnimationController

`animate(props, animationSettings?)` is the direct animation API. It bypasses the transition system and imperatively starts an animation.

```typescript
animate(
  props: Partial<INodeAnimateProps<CoreShaderNode>>,
  animationSettings?: AnimationSettings,
): IAnimationController {
  isDev && assertTruthy(this.rendered, 'Node must be rendered before animating');
  return (this.lng as IRendererNode).animate(
    props,
    animationSettings || this.animationSettings || {},
  );
}
```

Key behaviors:
- **Dev assertion**: throws if called before the node is rendered
- **Settings priority**: per-call settings > `this._animationSettings` > `Config.animationSettings` > `{}`
- **Returns**: `IAnimationController` from the Lightning renderer — supports `.start()`, `.stop()`, `.pause()`, `.waitUntilStopped()` (promise)

Unlike the `transition` system (which is declarative and triggers on property set), `animate()` is imperative — you control when it starts.

Common usage:
```typescript
node.animate({ x: 200, alpha: 0 }, { duration: 300 }).start()
// or wait for completion:
await node.animate({ alpha: 0 }, { duration: 300 }).start().waitUntilStopped()
```

The `chain()` method builds on this to sequence multiple animations. Since [[ElementNode chain and start methods enable sequenced animation queuing]], use chain/start for multi-step sequences.

---

Source: [[core-elementNode]]
Domains:
- [[core guide]]
- [[styling guide]]
