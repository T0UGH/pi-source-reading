# PI Session Runtime

## Core judgment
PI’s session runtime is built around **runtime replacement**, not transcript-only persistence.

Relevant files:
- `packages/coding-agent/src/core/agent-session-runtime.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/session-manager.ts`

## Session object vs runtime object
`AgentSession` handles one active session.
`AgentSessionRuntime` owns the current session plus its cwd-bound services and knows how to replace them.

This separation matters because switching, forking, importing, or reloading is not modeled as a small mutation on one long-lived object.
Instead, pi tears down the current runtime and creates a fresh one.

## Replacement lifecycle
The runtime replacement path is structurally:
1. emit before-event (`session_before_switch` or `session_before_fork`)
2. allow extensions to cancel
3. emit `session_shutdown`
4. dispose current session
5. create fresh services/session via `createRuntime(...)`
6. apply the new session/services
7. rebind upper layers through `rebindSession(...)`

## Why this matters
This design keeps cwd-bound state honest:
- resource discovery can change with cwd
- model/provider availability can change
- extension bindings must be rebuilt
- old session-bound references should not keep working silently

## Stale extension instances
During disposal, the extension runner is invalidated.
Old extension instances become explicitly stale and throw if reused.
This is a strong identity boundary: pi prefers hard failure over accidental cross-session leakage.

## Session persistence model
The session file is tree-structured JSONL, not just a flat chat transcript.
Entries include:
- messages
- model changes
- thinking-level changes
- compaction entries
- branch summaries
- custom entries
- labels
- session metadata

This is why `/tree`, `/fork`, and `/clone` are real runtime semantics rather than superficial UI features.
