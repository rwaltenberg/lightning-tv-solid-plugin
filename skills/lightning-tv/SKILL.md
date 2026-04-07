---
name: lightning-tv
description: "Lightning TV framework expertise for @lightningtv/solid v3.x Smart TV applications. Prevents DOM hallucination and provides a traversable knowledge graph of 173 atomic reference entries. Verified against @lightningtv/solid v3.2.3 source."
trigger: code imports `@lightningtv/solid` or `@lightningtv/solid/primitives`, or user asks about Lightning TV, Lightning renderer, or Smart TV UI development with SolidJS
user-invocable: true
---

# Lightning TV Project Detected (@lightningtv/solid v3.x)

You are working on a **@lightningtv/solid** application. This framework renders to a **WebGL/Canvas GPU scene graph**, NOT the browser DOM. Applying standard web patterns will break things silently.

> **Compatibility:** This knowledge base was extracted from `@lightningtv/solid` **v3.2.3** source code. It is valid for the **v3.x** major version line. APIs may differ in v2.x or earlier.

## Critical Rules (always apply)

1. **ONLY `<view>`, `<node>`, `<text>` exist.** No `<div>`, `<span>`, `<img>`, or any HTML tag.
2. **No DOM APIs.** No `document`, `addEventListener`, `querySelector`, `classList`, `innerHTML`.
3. **`forwardFocus` on every container** that holds focusable children -- without it, children never receive focus.
4. **Key handlers must `return true`** to stop event propagation.
5. **JSX inline props override `style` object.** Use signals on individual props or `$`-prefixed states for dynamic visuals.
6. **Always `color={0xffffffff}`** when using `src` for images.

## Before writing or reviewing @lightningtv/solid code

Read the full guardrails and quick reference:
- **Guardrails & rules**: `notes/guardrails.md` (in this skill's directory)
- **Knowledge graph hub**: `entries/index.md` (in this skill's directory)

## Knowledge Graph (173 entries across 9 domains)

The `entries/` directory contains a wiki-linked knowledge graph. Start from the guides:

| Guide | Domain |
|-------|--------|
| `entries/core guide.md` | ElementNode, renderer, config, task scheduling |
| `entries/styling guide.md` | Flex layout, states, transitions, shaders/effects |
| `entries/focus guide.md` | Focus manager, key handling, capture/bubble phases |
| `entries/navigation guide.md` | Row/Column nav, spatial nav, scrolling modes |
| `entries/components guide.md` | Row, Column, Grid, Virtual, Image, Lazy, and more |
| `entries/input guide.md` | useFocusManager, useHold, useMouse, key maps |
| `entries/accessibility guide.md` | Announcer, speech synthesis, ARIA labels |
| `entries/performance guide.md` | Virtual scrolling, lazy loading, preserve, task queue |
| `entries/routing guide.md` | HashRouter, KeepAlive, route preservation |

Each entry is titled as a prose statement (e.g., "Row component is a horizontal navigable list with built-in flex layout and 30px gap") and contains YAML frontmatter with `type` (api/pattern/gotcha/architecture/comparison) and `module` fields.

### Finding information

- **By domain**: Read the relevant guide file, which links to all entries with context phrases
- **By type**: Search for `type: gotcha` to find all pitfalls, `type: api` for API references
- **By keyword**: Search entry filenames or content for the component/concept name
- **By connection**: Follow `[[wiki links]]` between entries to traverse related concepts

## When dispatching subagents for implementation

Include these instructions in the subagent prompt:
- "This is a @lightningtv/solid project. Only `<view>`, `<node>`, `<text>` JSX elements exist. No HTML tags. No DOM APIs."
- "Read `[skill-dir]/notes/guardrails.md` before writing code."
- "For component API details, read the relevant entry in `[skill-dir]/entries/`."
