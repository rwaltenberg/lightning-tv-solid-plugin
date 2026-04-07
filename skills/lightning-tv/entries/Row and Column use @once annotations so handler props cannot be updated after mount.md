---
description: All Row and Column event handler props are annotated with @once at render time, freezing them permanently — prop changes after mount are silently ignored
type: gotcha
module: primitives
created: 2026-04-07
---

# Row and Column use @once annotations so handler props cannot be updated after mount

Row and Column apply `/* @once */` to all key handler and behavioral props. In Solid.js, `@once` tells the compiler to read the value once at render time and never set up a reactive subscription. This means:

```tsx
// This does NOT work — handler will never update:
const [myHandler, setMyHandler] = createSignal(() => false);
<Row onLeft={myHandler()} />

// Changing setMyHandler later has NO effect on the Row
```

Affected props in Row:
- `onLeft`, `onRight` — chained with internal handler at mount
- `onSelectedChanged` — chained with scrollRow at mount
- `onLayout` — chained with scrollRow at mount (only when `selected` is non-zero)
- `transition` — initialized to `{}`

Affected props in Column:
- `onUp`, `onDown` — chained with internal handler at mount
- `onSelectedChanged` — chained with scrollColumn at mount
- `onLayout` — chained with scrollColumn at mount (only when `selected` is non-zero)

The internal navigation handlers (`onLeft`/`onRight`/`onUp`/`onDown`) are module-level singletons — they don't need reactivity. But user-provided handlers are also frozen via the `chainFunctions` + `@once` combination.

**Workaround:** If you need dynamic behavior in a handler, read reactive signals from a closure captured at render time — the handler itself stays stable but its logic can read current reactive state:

```tsx
const [filter, setFilter] = createSignal('all');

<Row
  onLeft={() => {
    // This closure is fixed at mount, but filter() is read reactively each call
    if (filter() === 'all') return false;
    return false; // let default navigation proceed
  }}
/>
```

---

Source: [[primitives-Row]]
Domains:
- [[navigation guide]]
- [[components guide]]
