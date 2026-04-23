# PI Extension Runtime

## Core judgment
PI extensions use a two-phase model:
1. **load-time registration**
2. **run-time binding**

This is the key to understanding why extensions in pi are closer to host modules than ordinary plugins.

## Load-time registration
Relevant file: `packages/coding-agent/src/core/extensions/loader.ts`

During loading:
- pi creates a shared runtime stub with `createExtensionRuntime()`
- each extension factory gets an `ExtensionAPI`
- the extension factory is awaited, so async initialization is allowed
- registration methods populate extension-owned maps:
  - handlers
  - tools
  - commands
  - shortcuts
  - flags
  - message renderers

At this stage, most action methods are intentionally unavailable.
This means extensions can declare capabilities before a live session exists, but cannot yet act on a session.

## Deferred provider registration
A special case exists for `registerProvider()`.
Before runtime binding, provider registrations are queued into `pendingProviderRegistrations`.
After binding, these registrations are flushed into the active model registry.

This means provider registration is treated as a startup-time resource declaration, not just an ordinary session action.

## Run-time binding
Relevant files:
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`

When `AgentSession.bindExtensions()` runs, pi binds:
- core actions (`sendMessage`, `setModel`, `setActiveTools`, etc.)
- context actions (`abort`, `compact`, `getContextUsage`, etc.)
- command/session actions (`newSession`, `fork`, `switchSession`, `reload`, etc.)
- UI context for interactive surfaces

## What extensions can influence
Extensions can reach far beyond tool registration:
- mutate context before provider calls
- patch provider payloads
- mutate tool call arguments via `tool_call`
- patch tool results via `tool_result`
- cancel session switch/fork/compact/tree operations
- contribute additional skills/prompts/themes via `resources_discover`
- alter UI through widgets, footer/header, editor, overlays, and status hooks

## Main takeaway
PI’s extension system is effectively part of the host runtime.
It is designed to reshape runtime behavior, not just bolt on convenience features.
