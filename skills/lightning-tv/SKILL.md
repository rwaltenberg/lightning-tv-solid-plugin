---
name: lightning-tv
description: Lightning TV framework expertise for @lightningtv/solid Smart TV applications. Prevents DOM hallucination and enforces framework constraints.
trigger: code imports `@lightningtv/solid` or `@lightningtv/solid/primitives`, or user asks about Lightning TV, Lightning renderer, or Smart TV UI development with SolidJS
user-invocable: true
---

# Lightning TV Project Detected

You are working on a **@lightningtv/solid** application. This framework renders to a **WebGL/Canvas GPU scene graph**, NOT the browser DOM. Applying standard web patterns will break things silently.

## Critical Rules (always apply)

1. **ONLY `<view>`, `<node>`, `<text>` exist.** No `<div>`, `<span>`, `<img>`, or any HTML tag.
2. **No DOM APIs.** No `document`, `addEventListener`, `querySelector`, `classList`, `innerHTML`.
3. **`forwardFocus` on every container** that holds focusable children -- without it, children never receive focus.
4. **Key handlers must `return true`** to stop event propagation.
5. **`style` is set once and locked.** Use signals on individual props or `$`-prefixed states for dynamic visuals.
6. **Always `color={0xffffffff}`** when using `src` for images.

## Before writing or reviewing @lightningtv/solid code

Read the full guardrails and quick reference:
- **Guardrails & rules**: `notes/guardrails.md` (in this skill's directory)
- **Knowledge graph index**: `notes/MOC.md` (in this skill's directory)

## For API details on any component or utility

Read the relevant API note from `notes/api/` (in this skill's directory):
- `notes/api/MOC.md` lists every available reference
- Each note has full props, defaults, signatures, and gotchas

## When dispatching subagents for implementation

Include these instructions in the subagent prompt:
- "This is a @lightningtv/solid project. Only `<view>`, `<node>`, `<text>` JSX elements exist. No HTML tags. No DOM APIs."
- "Read `[skill-dir]/notes/guardrails.md` before writing code."
- "For component API details, read the relevant note in `[skill-dir]/notes/api/`."
