# 2026-04-24 pi ch03 subagent expansion v1

## 章节定位

第 03 章接在 session tree 后面。

第 02 章已经说明：`pi` 的 session 不是聊天记录，而是 runtime state。第 03 章要继续往上走一步：如果底层 session 是 runtime，那么高级工作流就不必都写死在 core product 里；它可以由 extension 调度出新的 runtime。

所以这一章不写“subagent 很酷”，也不写“多 agent 协作指南”。

它只证明一句话：

> `pi` 的 subagent 不是在主上下文里做 prompt 分身，而是由 extension 层发起的独立 runtime 委派。

这句话是全书主线的第三块证据：

1. 01 章：`pi` 更像可编程宿主；
2. 02 章：session tree 证明它把工作过程保存成 runtime state；
3. 03 章：subagent 证明高级工作流可以在 extension 层长出来。

---

## 正文应该回答的问题

### 1. 为什么不是 prompt 分身？

很多 subagent 方案的本质是：

- 主 agent 仍在同一个上下文里；
- 只是换一个 system prompt / persona；
- 或者让模型“假装”自己是 reviewer、planner、worker。

`pi` 的 subagent 不是这个路线。

源码里 `runSingleAgent()` 明确组装：

- `--mode json`
- `-p`
- `--no-session`
- 可选 `--model`
- 可选 `--tools`
- 可选 `--append-system-prompt <tmpfile>`

最后通过 `spawn(...)` 起一个新的 `pi` 进程。

这里最重要的是：

- `--no-session`：默认不共享主会话历史；
- `--append-system-prompt`：agent definition 可以真实注入子进程 prompt；
- `--tools` / `--model`：每个 agent 可以有独立工具和模型；
- `--mode json`：父进程消费的不是普通文本，而是结构化事件流。

所以正文要把判断写清楚：

> 这不是“同一个模型切换角色”，而是“宿主启动另一个 agent runtime”。

---

### 2. 为什么它仍然是 extension，而不是 core 内建工作流？

`subagent` 是通过 `pi.registerTool({ name: "subagent", ... })` 注册出来的。

这点很关键。

如果放在 Claude Code / cc 的产品路线里，subagent/fork 更像 core runtime 的一条优化路径。比如 cc 的 fork path 重点在：

- parent assistant message；
- byte-identical prefix；
- prompt cache sharing；
- synthetic fork child rules；
- 高性能地从主 agent 内部分叉 worker。

而 `pi` 这里不是把 subagent 做进 core，而是把“起新 pi、发现 agent、注入 prompt、消费 JSON、渲染结果”做成 extension 层的一套工具。

这恰好回到全书主线：

> `pi` 的高级工作流不是官方一次性替你定死，而是由宿主 runtime 暴露 surface，让用户在外层继续生长。

---

### 3. single / chain / parallel 说明什么？

`subagent` 示例不是只跑一个 worker。

它支持三种执行形态：

- single：一个 agent 执行一个 task；
- chain：按顺序执行，后一步可以引用 `{previous}`；
- parallel：并行多个任务，最多 8 个 task，并发上限 4。

这说明 `pi` 的 subagent 不是概念 demo，而是已经包含了多 agent orchestration 的最小骨架。

正文里可以这样写：

> 它没有把 orchestration 设计成一个官方工作流菜单，而是把最小调度器写成 extension：你可以从 single 起步，也可以把它组合成 chain 或 parallel。

这里要避免夸大：这不是说 `pi` 已经有成熟的多 agent 平台，而是说它证明了宿主路线的可行性。

---

### 4. project agents 的信任边界为什么值得写？

`agents.ts` 里 agent discovery 支持：

- user：`~/.pi/agent/agents/*.md`
- project：`.pi/agents/*.md`
- both

默认只加载 user-level agents。project-local agents 必须显式选择 scope；如果真的要跑 project agent，交互模式下还会提示确认。

这部分值得写进正文，因为它把“可编程宿主”的代价也讲出来了。

只要允许 repo 自带 agent prompt，就会出现信任边界问题：

- repo 可以影响 agent 行为；
- agent 可能被授予读文件、跑命令等能力；
- project-local prompt 不应默认可信。

所以 `pi` 的设计不是无脑开放，而是在 extension 能力和安全治理之间做了一个最小边界。

这也能自然接到第 05 章：host/runtime 路线的代价是治理复杂度上升。

---

## 建议正文结构

### 标题

03｜pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派

### 开头

从读者常见误解切入：看到 subagent，容易以为就是多个 prompt 角色。

然后直接给判断：`pi` 的实现不是角色扮演，而是 extension spawn 独立 `pi` runtime。

### 1. subagent 首先是一个 extension tool

讲 `pi.registerTool({ name: "subagent" })`。

强调：高级工作流在 extension 层，而不是 core 内建菜单。

### 2. 真正的隔离来自新进程和 `--no-session`

讲 `runSingleAgent()` 的 CLI 参数组装。

重点解释：独立进程、独立上下文、agent-specific prompt / tools / model。

### 3. 父进程消费的是结构化事件流

讲 `--mode json`、逐行 `JSON.parse`、`message_end`、`tool_result_end`、usage 聚合。

这里可以把“子 agent 返回文本”与“子 runtime 回传事件”区别开。

### 4. single / chain / parallel 是最小 orchestration 骨架

讲三种模式，说明 extension 层已经能长出调度器。

### 5. project agents 暴露了宿主路线的信任边界

讲 user/project/both、默认 user、确认 project agents。

### 6. 和 cc fork path 的差异

收束为对照：

- cc：core 内部的优化型分叉机制；
- pi：extension 层的可编排 runtime 委派。

### 结尾

回到全书主线：

> 如果 session tree 说明 `pi` 把工作过程保存成 runtime state，那么 subagent 就说明：`pi` 也在让新的工作流从 runtime surface 上长出来。

---

## 建议配图

文件：`docs/assets/pi-ch03-subagent-runtime-delegation.svg`

图要表达三层：

1. 主 `pi` runtime：当前会话里调用 subagent tool；
2. extension 调度器：发现 agents，选择 single/chain/parallel，组装 CLI 参数；
3. 子 `pi` runtime：独立进程、独立上下文、JSON event stream 回传。

底部结论：

> pi 让多 agent orchestration 作为宿主能力长出来。

---

## 最该避免的写法

1. 不要写成“pi 支持 subagent，所以很强”。太泛。
2. 不要写成“多 agent 教程”。这不是本书目标。
3. 不要把 `pi` 夸成成熟多 agent 平台。现在更准确的说法是：它给出了 extension 层长出 orchestration 的最小骨架。
4. 不要只写 spawn 进程。关键是：spawn + prompt/tool/model 注入 + JSON event stream + UI/usage 回传 + project agent trust boundary。
