# Lightning TV Expert Plugin

A Claude Code plugin that provides framework expertise for building Smart TV applications with [`@lightningtv/solid`](https://github.com/niceDev0908/solid).

## How It Works

The plugin installs a **skill** that:

1. **Auto-triggers** when your codebase imports `@lightningtv/solid`
2. **Injects lightweight guardrails** (~300 tokens) into the conversation -- the 6 most critical rules that prevent DOM hallucination
3. **Directs Claude to read detailed references on-demand** from a knowledge graph of 80 atomic notes

This means it works seamlessly with any workflow -- including [superpowers](https://github.com/obra/superpowers)' subagent-driven development, brainstorming, and TDD skills. The controller receives the framework constraints and passes them to any subagents it spawns.

## Architecture

```
skills/lightning-tv/
  SKILL.md                    # ~300 tokens: trigger + critical rules + read instructions
  notes/
    guardrails.md             # ~800 tokens: full forbidden tags, DOM APIs, 20-point reference
    MOC.md                    # Master index of all knowledge domains
    constraints/  (8 notes)   # Anti-patterns that crash or fail silently
    core/         (6 notes)   # ElementNode, rendering pipeline, lifecycle
    flex/        (15 notes)   # Flex engine traps and CSS divergences
    focus/       (15 notes)   # D-pad navigation, key events, focus tree
    reactivity/   (7 notes)   # SolidJS signals, styles, states, transitions
    api/         (29 notes)   # Full API reference for every primitive
```

**Token flow**: trigger costs ~300. First code task reads guardrails (~800). Each component adds ~300-500 for its API note. Typical conversation: ~1,400 tokens instead of a monolithic ~7,500.

## Installation

```bash
# Via plugin command
/plugin install lightning-tv

# Or add marketplace
/plugin marketplace add your-org/lightning-tv-agent
```

## Manual Invocation

Type `/lightning-tv` to activate the skill explicitly in any conversation.

## Key Rules Enforced

1. Only `<view>`, `<node>`, `<text>` exist -- no HTML tags
2. No DOM APIs -- no `document`, `addEventListener`, `querySelector`
3. `forwardFocus` required on every container with focusable children
4. Key handlers must `return true` to stop event propagation
5. `style` is set once and locked -- use signals or states for dynamics
6. Always set `color={0xffffffff}` when using `src` for images

See `skills/lightning-tv/notes/guardrails.md` for the complete 20-point ruleset.

## Works With Superpowers

When used alongside the [superpowers](https://github.com/obra/superpowers) plugin:

- **Brainstorming**: The skill's guardrails inform design decisions
- **Subagent-driven development**: The controller includes Lightning TV constraints when dispatching implementer subagents
- **Code review**: Reviewers can consult the knowledge graph for framework-specific checks
- **TDD**: Test implementations respect the framework's actual API surface

No special configuration needed -- the skill auto-triggers and integrates naturally.

## Source Material

All notes were extracted from 5 architectural documents covering the `@lightningtv/solid` framework and verified against sources with zero critical issues.
