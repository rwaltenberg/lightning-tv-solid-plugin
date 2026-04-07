# Lightning TV Expert Plugin

A Claude Code plugin that provides framework expertise for building Smart TV applications with [`@lightningtv/solid`](https://github.com/lightning-tv/solid) **v3.x**.

> **Compatibility:** All knowledge was extracted from `@lightningtv/solid` **v3.2.3** source code and is valid for the **v3.x** major version line. APIs may differ in v2.x or earlier.

## How It Works

The plugin installs a **skill** that:

1. **Auto-triggers** when your codebase imports `@lightningtv/solid`
2. **Injects lightweight guardrails** (~300 tokens) into the conversation -- the 6 most critical rules that prevent DOM hallucination
3. **Directs Claude to read detailed references on-demand** from a knowledge graph of 173 atomic entries across 9 domains

Because it's a skill (not an agent), it integrates seamlessly with any workflow -- including subagent-driven development, brainstorming, and TDD. The controller receives the framework constraints and passes them to any subagents it spawns.

## Architecture

```
skills/lightning-tv/
  SKILL.md                    # ~300 tokens: trigger + critical rules + read instructions
  notes/
    guardrails.md             # ~800 tokens: full forbidden tags, DOM APIs, 22-point ruleset
  entries/
    index.md                  # Hub: entry point to the knowledge graph
    core guide.md             # ElementNode, renderer, config, task scheduling
    styling guide.md          # Flex layout, states, transitions, shaders/effects
    focus guide.md            # Focus manager, key handling, capture/bubble
    navigation guide.md       # Row/Column nav, spatial nav, scrolling modes
    components guide.md       # Row, Column, Grid, Virtual, Image, Lazy, etc.
    input guide.md            # useFocusManager, useHold, useMouse, key maps
    accessibility guide.md    # Announcer, speech synthesis, ARIA
    performance guide.md      # Virtual scrolling, lazy loading, preserve, task queue
    routing guide.md          # HashRouter, KeepAlive, route preservation
    *.md                      # 173 atomic entries (api, pattern, gotcha, architecture)
```

**Token flow**: trigger costs ~300. First code task reads guardrails (~800). Each domain guide adds ~500-800. Individual entries are ~300-600 each. Claude reads only what's relevant to the current task.

## Knowledge Graph

Every entry is titled as a prose statement and linked via `[[wiki links]]`:

- **77 API entries** -- function signatures, parameters, return values, side effects
- **43 architecture entries** -- how systems work internally, design decisions, data flow
- **34 gotcha entries** -- misleading behaviors, undocumented quirks, silent failures
- **17 pattern entries** -- how to correctly use components and APIs
- **2 comparison entries** -- when to use X vs Y, trade-offs

All entries were extracted directly from `@lightningtv/solid` v3.2.3 source code (55 files) and cross-referenced across 9 domains.

## Installation

```bash
# Via plugin command
/plugin install lightning-tv

# Or add from marketplace
/plugin marketplace add your-org/lightning-tv-agent
```

## Manual Invocation

Type `/lightning-tv` to activate the skill explicitly in any conversation.

## Key Rules Enforced

1. Only `<view>`, `<node>`, `<text>` exist -- no HTML tags
2. No DOM APIs -- no `document`, `addEventListener`, `querySelector`
3. `forwardFocus` required on every container with focusable children
4. Key handlers must `return true` to stop event propagation
5. JSX inline props override `style` object -- use signals or states for dynamics
6. Always set `color={0xffffffff}` when using `src` for images

See `skills/lightning-tv/notes/guardrails.md` for the complete 22-point ruleset.

## Framework Version

| Field | Value |
|-------|-------|
| Framework | `@lightningtv/solid` |
| Version extracted from | **v3.2.3** |
| Compatible major line | **v3.x** |
| Renderer peer dependency | `@lightningjs/renderer ^3.0.0` |
| SolidJS peer dependency | `solid-js *` |
| Router peer dependency | `@solidjs/router ^0.15.1` |

## Compatibility

The skill-based architecture means it works with any Claude Code workflow or plugin. Because the guardrails are injected into the conversation context (not isolated in a subagent), any workflow that dispatches subagents will naturally pass the framework constraints along.

## Source Material

All 173 entries were extracted from the `@lightningtv/solid` v3.2.3 framework source code (55 files), verified against source with cross-referencing across all 9 domains, and checked against a prior v2.0.0 plugin for accuracy (4 incorrect rules corrected, 7 missing gotchas added).
