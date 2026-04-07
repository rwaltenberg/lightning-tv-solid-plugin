---
description: Accepts N functions (or falsy values) and returns a single function calling each in order; returning true from any function stops the chain, matching Lightning's key event propagation stop semantics
type: api
module: primitives
created: 2026-04-07
---

# chainFunctions composes event handlers where returning true stops the chain

`chainFunctions` is the standard utility for composing multiple Lightning TV event handlers without losing existing handlers. It is used throughout the primitives layer to allow user-provided callbacks to coexist with internal component logic.

```ts
import { chainFunctions } from '@lightningtv/solid/primitives';

function Button(props: NodeProps) {
  function onEnter(el: ElementNode) {
    // internal handler
  }
  return <view onEnter={chainFunctions(props.onEnter, onEnter)} />
}
```

**Behavior**:
- Falsy values (`undefined`, `null`, `false`) are filtered out
- If no functions remain: returns `undefined`
- If one function: returns it directly (no wrapper overhead)
- If multiple: returns a wrapper that calls each in sequence

**Stop behavior**: if any function returns `true`, the wrapper returns `true` immediately and stops. This mirrors how [[KeyHandlerReturn true stops key event propagation up the focus chain]] works in Lightning's key system.

**Order matters**: functions execute in argument order. Place user handlers first (`props.onEnter`) if user should be able to stop the chain before internal logic runs. Place internal handlers first if they should always run.

**`chainRefs`**: same function, typed for ref forwarding:
```ts
let localRef: ElementNode | undefined;
<view ref={chainRefs(props.ref, el => localRef = el)} />
```

---

Source: [[primitives-utils-chainFunctions]]
Domains:
- [[components guide]]
