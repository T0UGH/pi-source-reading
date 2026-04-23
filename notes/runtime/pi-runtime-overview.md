# PI Runtime Overview

## Core judgment
`pi-mono` is best understood as an extensible coding-agent host, not just a coding CLI and not just a backend orchestration engine.

## Why
The repo centers on `packages/coding-agent`, but that package sits on top of a broader stack:
- `packages/ai` — unified multi-provider LLM layer
- `packages/agent` — stateful agent loop and tool execution
- `packages/tui` — terminal UI primitives
- `packages/coding-agent` — interactive host and session product layer
- `packages/web-ui` — browser host surface
- `packages/mom` — Slack surface
- `packages/pods` — self-hosted deployment surface

## Stable architectural split
1. **provider/model layer** — model registry, auth, provider integration
2. **agent core** — message loop, tool execution, streaming, context transforms
3. **host/session layer** — session tree, branching, compaction, commands, runtime replacement
4. **extension layer** — tools, commands, shortcuts, provider injection, UI hooks
5. **surface layer** — interactive TUI, print mode, RPC mode, web UI, Slack bot

## Consequence
The most important thing about pi is not a single default workflow.
Its main value is that it exposes a programmable host that can be reshaped through extensions, skills, prompt templates, packages, and alternate frontends.
