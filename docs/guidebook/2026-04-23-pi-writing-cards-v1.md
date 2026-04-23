# 2026-04-23 pi writing cards v1

## Card 01

### 标题
pi 不是另一个 Claude Code，而是一个可编程的 coding-agent 宿主

### 核心问题
`pi` 最值得研究的地方到底是什么？

### 一句话 thesis
`pi` 的核心价值，不在于又做了一个 terminal coding agent，而在于它把 extension runtime 做成了系统中心，把高级工作流从 core 里外化出去。

### 关键源码锚点
- `packages/coding-agent/README.md`
- `packages/coding-agent/examples/extensions/README.md`
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/core/extensions/loader.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`
- 对照：
  - `cc/src/tools/AgentTool/builtInAgents.ts`
  - `cc/src/tools/AgentTool/loadAgentsDir.ts`

## Card 02

### 标题
为什么 pi 要把 session 做成树，而不是聊天记录

### 一句话 thesis
`pi` 把 session 做成 `id + parentId` 的树形 runtime，这让 `/tree`、`/fork`、`/clone`、branch summary、compaction 都变成底层自然能力，而不是 UI 花活。

### 关键源码锚点
- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

## Card 03

### 标题
pi 的 subagent 不是角色扮演，而是独立 runtime 的委派

### 一句话 thesis
`pi` 的 subagent 示例并不是在主会话里模拟多个角色，而是由 extension spawn 独立 `pi` 进程，用独立上下文窗口执行任务，再把结构化结果带回主会话。

### 关键源码锚点
- `packages/coding-agent/examples/extensions/subagent/README.md`
- `packages/coding-agent/examples/extensions/subagent/index.ts`
- `packages/coding-agent/examples/extensions/subagent/agents.ts`
- 对照：
  - `cc/src/tools/AgentTool/forkSubagent.ts`

## Card 04

### 标题
当 extension 能改 tool、UI、provider、compaction，agent 产品会变成什么

### 一句话 thesis
当 tool、UI、provider、compaction、session actions 都变成 extension runtime 的一部分时，agent 产品会从“固定功能集合”转向“可编排宿主”，但代价是治理复杂度和用户侧工程能力要求上升。

### 关键源码锚点
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/examples/extensions/plan-mode/`
- `packages/coding-agent/examples/extensions/custom-provider-*`

## 当前拍板

这本书不追求大全。

全书只服务一个总论点：

> `pi` 的卖点，不是它比 cc 多了什么功能，而是它把 agent 做成了可编程宿主。
