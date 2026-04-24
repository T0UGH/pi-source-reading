# 2026-04-23 pi book outline v1

## 总定位

这本书不是 `pi` 的完整说明书。

它要做的不是：
- 教你怎么安装
- 教你怎么切模型
- 教你所有命令怎么用
- 把 monorepo 每个 package 都讲一遍

它要做的是：

> **把 `pi` 讲成一条和 Claude Code 明显不同的 agent 产品路线。**

也就是说，这本书只关心：
- `pi` 和 cc 最大的差异点是什么
- `pi` 的真正卖点是什么
- 为什么这种路线对会自己搭 agent 工作台的人特别值钱

---

## 总论点

整本书只证明一句话：

> **Claude Code 更像一个已经做得很强的官方 agent 成品，而 `pi` 更像一个让你自己长出 agent 工作流的宿主。**

后面所有章节都围绕这句话服务。

---

## 我想写什么

我想把这本书先写成一册很集中的小书，结构控制在：

- 1 篇总入口
- 4 篇主章节
- 1 篇结语/路线总结

不是为了凑章节数，而是为了把最重要的差异点讲透。

---

# 目录草案

## 00 总入口
# 这不是另一个 Claude Code：为什么 pi 值得单独研究

### 这章回答什么
- 为什么 `pi` 不能按“又一个 coding agent”去读
- 为什么这本书不做大全
- 为什么“强成品 vs 可编程宿主”是正确入口

### 我想写的重点
- 先把读者从功能视角拽出来
- 明确这本书是产品路线导读，不是功能说明书
- 给出后面几章的阅读顺序

### 主要材料
- `packages/coding-agent/README.md`
- 当前 `docs/guidebook/scope` 文档

---

## 01
# pi 不是强成品 agent，而是可编程宿主

### 这章回答什么
- `pi` 和 cc 最大的差异点到底是什么
- 为什么 cc 更像“内建工作流优先”
- 为什么 `pi` 更像“宿主能力优先”

### 我想写的重点
这是整本书最重要的一章。

要讲清楚：
- cc 的高级工作流大多是 core product 的一部分
- `pi` 则把高级工作流尽量外化到 extension / package
- `pi` 的核心价值，不是再造一个 assistant，而是提供一套可以继续长工作流的 host runtime

### 主要材料
- `packages/coding-agent/README.md`
- `packages/coding-agent/examples/extensions/README.md`
- `packages/coding-agent/src/core/extensions/{types,loader,runner}.ts`
- 对照：
  - `cc/src/tools/AgentTool/builtInAgents.ts`
  - `cc/src/tools/AgentTool/loadAgentsDir.ts`

### 当前已有材料
- `docs/notes/2026-04-23-pi-vs-cc-差异点初判.md`
- `docs/notes/2026-04-23-pi-extension-runtime-初读.md`
- `docs/articles/2026-04-23-pi-不是强成品-agent-而是可编程宿主-v1.md`
- `docs/guidebook/2026-04-23-pi-ch01-expansion-v1.md`

---

## 02
# 为什么 pi 要把 session 做成树，而不是聊天记录

### 这章回答什么
- 为什么 tree session 是宿主路线的真正底座
- 为什么 `/tree` / `/fork` / `/clone` 在 `pi` 里不是 UI 花活
- 为什么这种 session runtime 更适合长期 agent 工作台

### 我想写的重点
这章要把“宿主不是空理念”讲实。

要讲清楚：
- `id + parentId` 的树形 session 结构
- `buildSessionContext()` 怎么从 leaf 回溯并重建当前 branch context
- compaction / branch summary / custom entries 怎么自然接进来
- session replacement / stale runtime 为什么是宿主路线必须面对的工程问题

### 主要材料
- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

### 当前已有材料
- `docs/notes/2026-04-23-pi-session-runtime-初读.md`
- `docs/guidebook/2026-04-24-pi-ch02-session-tree-expansion-v1.md`
- `docs/articles/2026-04-24-pi-session-tree-不是聊天记录而是-runtime-v1.md`
- `docs/assets/pi-ch02-session-tree-runtime.svg`

---

## 03
# pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派

### 这章回答什么
- `pi` 的 subagent 到底是 prompt 技巧，还是 runtime 级隔离
- 为什么它坚持用独立 `pi` 进程来做子 agent
- 这条路线和 cc 的 fork path 差在哪

### 我想写的重点
这章不是讲“subagent 很酷”，而是讲：

- 为什么 `pi` 把委派做成 extension 层的调度器
- 为什么它默认走独立进程、独立上下文
- agent-specific prompt / tool / model 是怎么注入进去的
- project-local agents 的信任边界为什么重要

### 主要材料
- `packages/coding-agent/examples/extensions/subagent/index.ts`
- `packages/coding-agent/examples/extensions/subagent/agents.ts`
- `packages/coding-agent/examples/extensions/subagent/README.md`
- 对照：
  - `cc/src/tools/AgentTool/forkSubagent.ts`

### 当前已有材料
- `docs/notes/2026-04-23-pi-subagent-实现细读-v1.md`

---

## 04
# plan mode 不是内建角色，而是宿主切出来的一种工作模式

### 这章回答什么
- 为什么 `pi` 没把 plan 做成 built-in worker
- 为什么它更愿意把 plan 做成 extension 里的双态 runtime
- 这说明 `pi` 的宿主路线已经走到多深

### 我想写的重点
这章要解释一个非常关键的区别：

- cc 的 plan 更像 built-in worker / official workflow
- `pi` 的 plan mode 更像宿主里的 mode switch

重点讲：
- 只读工具池
- bash allowlist
- `before_agent_start` 注入计划上下文
- `[DONE:n]` 进度机制
- custom entries + session restore

这章写好了，读者会更清楚：

> `pi` 不是只有一种外化路线（subagent），而是已经展示了至少两种：委派型和模态切换型。

### 主要材料
- `packages/coding-agent/examples/extensions/plan-mode/index.ts`
- `packages/coding-agent/examples/extensions/plan-mode/utils.ts`
- `packages/coding-agent/examples/extensions/plan-mode/README.md`
- 对照：
  - `cc/src/tools/AgentTool/builtInAgents.ts`

### 当前已有材料
- `docs/notes/2026-04-23-pi-plan-mode-实现细读-v1.md`

---

## 05
# 当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么

### 这章回答什么
- 如果一个 agent 把高级能力都外化到 runtime surface，会获得什么、失去什么
- `pi` 的真正卖点为什么不是单点 feature，而是整套宿主能力
- 这条路线的代价是什么

### 我想写的重点
这是全书的收口篇。

我要讲：
- `pi` 的 extension surface 已经覆盖到哪些层
- 这为什么意味着它更像 host / platform
- 这条路线为什么对系统型用户特别值钱
- 代价也要说透：治理、权限、复杂度、用户工程能力要求

### 主要材料
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/examples/extensions/custom-provider-*`
- 前面 subagent / plan mode 两章的结论

### 当前已有材料
- `docs/notes/2026-04-23-pi-extension-runtime-初读.md`
- 前面几章的细读笔记

---

## 06 结语
# 为什么 pi 代表另一种 agent 产品路线

### 这章回答什么
- 读完整本书后，应该怎样重新定位 `pi`
- 它不适合谁、又特别适合谁
- 它和 cc / Codex / 其他 agent 产品的关系应该怎么理解

### 我想写的重点
把全书收成一个更清楚的判断：

- 它不是为了所有人
- 它不是功能最全就赢
- 它的价值在于：把 agent 从强成品反过来做成宿主

这章不需要很长，但要够稳。

---

## 当前写作顺序建议

我建议按这个顺序推进：

1. 先把 01 打磨稳
2. 再写 02 session tree
3. 再写 03 subagent
4. 再写 04 plan mode
5. 最后写 05 收口篇
6. 再回头补 00 和 06

原因很简单：
- 01 先定总论点
- 02 给宿主底座
- 03/04 给两种工作流外化路线
- 05 再收成完整产品判断

---

## 当前拍板

如果只保留一句，这本书现在最该写的是：

> **`pi` 的卖点，不是它比 cc 多了什么功能，而是它把 agent 做成了可编程宿主。**

后面的所有章节都只是在证明这句话。