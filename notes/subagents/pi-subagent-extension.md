# PI Subagent Extension

## Core judgment
PI’s showcased subagent pattern is **not** a built-in team runtime.
It is an extension-level orchestration tool that spawns separate `pi` subprocesses.

Relevant files:
- `packages/coding-agent/examples/extensions/subagent/index.ts`
- `packages/coding-agent/examples/extensions/subagent/agents.ts`

## Main mechanism
Each delegated worker is started as a separate process using:
- `spawn(...)`
- `pi --mode json -p --no-session`

Optional flags are added per agent definition:
- `--model`
- `--tools`
- `--append-system-prompt <tmpfile>`

This means the worker is not a hidden in-process subordinate runtime.
It is a separate short-lived pi invocation with isolated context.

## Supported orchestration modes
The example tool implements three modes:
- **single** — one agent, one task
- **parallel** — many tasks with bounded concurrency
- **chain** — sequential steps with `{previous}` placeholder substitution

Hard limits in the example:
- `MAX_PARALLEL_TASKS = 8`
- `MAX_CONCURRENCY = 4`

## Output model
The parent tool consumes JSON-mode events from child stdout and reconstructs:
- assistant messages
- tool result messages
- usage stats
- stop reason
- error message
- final text output

So the parent is not just shelling out and grabbing plain text. It is rebuilding a structured result object.

## Agent definition model
Agents are discovered from markdown files:
- `~/.pi/agent/agents/*.md`
- `.pi/agents/*.md`

Frontmatter fields are intentionally light:
- `name`
- `description`
- `tools`
- `model`

The markdown body becomes the worker system prompt.

## Security boundary
Project-local agents are treated as higher risk because they are repo-controlled prompts.
When project agents are enabled, the example can require explicit UI confirmation before running them.

## Main takeaway
PI’s multi-agent story is host-extensible rather than runtime-native.
The official example demonstrates that subagents are something you can grow from the host, not a mandatory built-in orchestration core.
