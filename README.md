# Lightning TV Expert Agent

A Claude Code plugin providing an expert AI coding agent for building Smart TV applications with the [`@lightningtv/solid`](https://github.com/niceDev0908/solid) framework.

## What It Does

This plugin installs a specialized agent that:

- **Prevents DOM hallucination** -- enforces that only `<view>`, `<node>`, and `<text>` JSX elements exist (no HTML tags, no DOM APIs)
- **Knows the custom flex engine** -- 15 documented traps where Lightning's layout diverges from CSS Flexbox
- **Handles D-pad navigation** -- 15 focus/navigation rules for TV remote control input
- **Provides full API references** -- 29 API notes covering every primitive (Row, Column, Grid, VirtualRow, Image, Marquee, etc.) with complete props, defaults, and behavioral details

## Architecture

The agent uses an Ars Contexta-style knowledge graph instead of a monolithic system prompt:

- **`AGENT.md`** (~1,000 tokens, always loaded) -- guardrails, forbidden element lists, 20-point quick reference
- **`notes/`** (80 atomic notes, loaded on-demand) -- deep technical reference organized by domain

```
agents/lightning-tv/
  AGENT.md
  notes/
    MOC.md                    # Master index
    constraints/  (8 notes)   # Anti-patterns that crash or fail silently
    core/         (6 notes)   # ElementNode, rendering pipeline, lifecycle
    flex/        (15 notes)   # Flex engine traps and CSS divergences
    focus/       (15 notes)   # D-pad navigation, key events, focus tree
    reactivity/   (7 notes)   # SolidJS signals, styles, states, transitions
    api/         (29 notes)   # Full API reference for every primitive
```

This gives the agent **5x more knowledge** than a monolithic prompt while consuming **86% fewer tokens** on every request.

## Installation

```bash
# From GitHub
/plugin marketplace add your-org/lightning-tv-agent

# Or locally
claude --plugin-dir ./lightning-tv-agent
```

## Key Rules Enforced

1. Only `<view>`, `<node>`, and `<text>` exist -- no HTML tags
2. `forwardFocus` required on every container with focusable children
3. `flexGrow` requires >= 2 children (silently ignored with 1)
4. `style` is set once and locked -- use signals or states for dynamic visuals
5. Key handlers must `return true` to stop event propagation
6. Always set `color={0xffffffff}` when using `src` for images
7. `textAlign` requires `contain` -- silently ignored without it
8. Use `Visible` over `Show` for Lightning nodes that toggle frequently
9. Virtual component children receive Accessors -- call `item()`, not `item`
10. Never call DOM APIs -- no `document`, `addEventListener`, `querySelector`

See `agents/lightning-tv/AGENT.md` for the complete 20-point ruleset.

## Source Material

All notes were extracted from 5 architectural documents covering the `@lightningtv/solid` framework and verified against sources with zero critical issues.
