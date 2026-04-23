# 2026-04-23 pi guidebook scope v1

## 定位

这本 `pi` 导读书不做大而全。

不写成：
- 功能手册
- 命令大全
- provider 大全
- TUI 教程
- monorepo 全量漫游

而是写成一本：

> **说明 `pi` 和 Claude Code 最大差异点，以及 `pi` 这条产品路线为什么值得研究的小书。**

## 只回答三个问题

### Q1
`pi` 和 Claude Code 最大的差异点到底是什么？

### Q2
`pi` 真正的卖点是什么？

### Q3
为什么它对会自己搭 agent 工作台的人特别值得研究？

## 小书形态

### 00 总入口
这不是另一个 Claude Code：为什么 pi 值得单独研究

### 01
pi 不是强成品 agent，而是可编程宿主

### 02
为什么 pi 要把 session 做成树，而不是聊天记录

### 03
pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派

### 04
当 tool、UI、provider、compaction 都变成 extension surface，agent 产品会变成什么

## 总论点

整本书只证明一句话：

> Claude Code 更像一个已经做得很强的官方 agent 成品，而 pi 更像一个让你自己长出 agent 工作流的宿主。

## 当前源码重点

### 01 主要看
- `packages/coding-agent/README.md`
- `packages/coding-agent/examples/extensions/README.md`
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/core/extensions/loader.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`
- 对照：
  - `cc/src/tools/AgentTool/builtInAgents.ts`
  - `cc/src/tools/AgentTool/loadAgentsDir.ts`

### 02 主要看
- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

### 03 主要看
- `packages/coding-agent/examples/extensions/subagent/index.ts`
- `packages/coding-agent/examples/extensions/subagent/agents.ts`
- `packages/coding-agent/examples/extensions/subagent/README.md`
- 对照：
  - `cc/src/tools/AgentTool/forkSubagent.ts`
