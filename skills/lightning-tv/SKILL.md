---
name: lightning-tv
description: Activate the Lightning TV expert agent for building Smart TV applications with @lightningtv/solid. Delegates coding tasks to a specialized agent that enforces framework constraints and has a traversable knowledge graph of 80 API/rule reference notes.
trigger: code imports `@lightningtv/solid` or `@lightningtv/solid/primitives`, or user asks about Lightning TV, Lightning renderer, or Smart TV UI development with SolidJS
user-invocable: true
---

# Lightning TV Expert Mode

You are now working on a **@lightningtv/solid** Smart TV application. This framework renders to a WebGL/Canvas GPU scene graph -- NOT the browser DOM.

**Delegate all coding tasks to the `lightning-tv` agent** using the Agent tool with `subagent_type: "lightning-tv"`. The agent has:
- Anti-hallucination guardrails (forbidden HTML tags, forbidden DOM APIs)
- 80 atomic reference notes covering flex layout traps, focus/navigation rules, reactivity patterns, and full API docs for every primitive
- A traversable knowledge graph it consults before writing code

## When to delegate

- Writing or modifying components (`<view>`, `<text>`, Row, Column, Grid, etc.)
- Debugging layout, focus, or navigation issues
- Questions about API props, defaults, or behavioral quirks
- Code review of @lightningtv/solid code

## When NOT to delegate

- Build tooling, bundler config, CI/CD (not framework-specific)
- Pure SolidJS logic with no Lightning TV rendering (signals, stores, routing)
- General TypeScript/JavaScript questions

## Quick guardrails (always apply, even without delegation)

1. **ONLY `<view>`, `<node>`, `<text>` exist** -- no HTML tags
2. **No DOM APIs** -- no `document`, `addEventListener`, `querySelector`
3. **`forwardFocus` on every container** with focusable children
4. **Key handlers must `return true`** to stop propagation
5. **`style` is set once and locked** -- use signals or states for dynamics
6. **Always `color={0xffffffff}`** when using `src` for images
