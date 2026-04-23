# 2026-04-23 pi subagent 实现细读 v1

## 结论

`pi` 的 subagent 路线不是一句“spawn 一个新进程”就能概括完。

从 `packages/coding-agent/examples/extensions/subagent/index.ts` 这轮细读看，它已经形成了一套比较完整的宿主实现：

- 独立 `pi` 进程
- 独立上下文窗口
- 可注入 agent-specific system prompt
- 使用 JSON mode 捕获结构化事件
- 支持 single / chain / parallel 三种执行形态
- 明确区分 user agents 与 project agents
- 对 project-local agents 有显式信任确认

一句话说：

> `pi` 的 subagent 不是 prompt 分身，而是宿主 runtime 之上的受控委派器。`

---

## 关键锚点

- `packages/coding-agent/examples/extensions/subagent/index.ts`
- `packages/coding-agent/examples/extensions/subagent/agents.ts`
- `packages/coding-agent/examples/extensions/subagent/README.md`
- 对照：`cc/src/tools/AgentTool/forkSubagent.ts`

---

## 这轮看到的关键实现点

## 1. 真正的隔离不是“换个 prompt”，而是直接起新 `pi`

在 `runSingleAgent()` 里，subagent 执行主链非常明确：

1. 找到 agent definition
2. 组装 `pi` CLI 参数：
   - `--mode json`
   - `-p`
   - `--no-session`
   - 可选 `--model`
   - 可选 `--tools`
3. 如果 agent 自己有 `systemPrompt`，先写到临时文件
4. 再用 `--append-system-prompt <tmpfile>` 注入
5. 最终 `spawn(...)` 一个新的 `pi` 进程执行任务

这里最关键的点有两个：

### A. `--no-session`
说明子 agent 默认不是共享主会话历史，而是一次性独立运行体。

### B. `--append-system-prompt`
说明 agent definition 并不是只作为调度标签，而是真的能把自己的角色 prompt 注入到子进程。

也就是说：

> subagent 在 `pi` 这里已经是“新 runtime + 新 prompt layer + 新工具集合”的组合体。

---

## 2. 它不是把 stdout 当普通文本，而是把子 agent 当事件流来消费

`runSingleAgent()` 里不是简单读最终文本，而是：

- 按行处理子进程 stdout
- 每行尝试 `JSON.parse`
- 识别：
  - `message_end`
  - `tool_result_end`
- 把这些事件重新积累为 `messages`
- 同时累计 usage：
  - input/output
  - cacheRead/cacheWrite
  - cost
  - contextTokens
  - turns

这很重要，因为这说明：

> `pi` 的 subagent 不是 shelling out 一个脚本，而是在消费另一个 agent runtime 的结构化事件流。

所以主会话能在 UI 里展示：
- tool call
- 结果
- token/cost/context usage
- 最终输出

这个设计比“子 agent 返回一段文本”成熟很多。

---

## 3. single / chain / parallel 不是概念层，而是真做进 extension 里的执行模型

这套 subagent 工具不是只支持一个 worker。

它明确支持三种模式：

### single
- 一个 agent
- 一个 task

### chain
- 顺序执行
- 后一步可以引用前一步输出：`{previous}`
- 任一步失败则整条链停止

### parallel
- 一次多个 task
- `MAX_PARALLEL_TASKS = 8`
- `MAX_CONCURRENCY = 4`
- 主会话会持续收到 streaming updates

这意味着：

> `pi` 已经把多 agent orchestration 的最小执行骨架做出来了，只是把它放在 extension 层，而不是 core。`

这正好体现它的产品哲学：
- 不把高级工作流内建进产品主干
- 但宿主能力足够强，外面可以长出真正能跑的 orchestrator

---

## 4. agentScope 是一条很明确的信任边界

`agents.ts` 里 agent 发现逻辑也很值得注意。

它支持：
- `user`
- `project`
- `both`

来源分别是：
- `~/.pi/agent/agents/*.md`
- `.pi/agents/*.md`

而且：
- 默认只加载 user-level agents
- project-local agents 必须显式开 scope
- 若启用 project/both，且当前请求确实会跑 project agent，还会弹确认

这个设计说明作者非常清楚：

> repo 里的 agent prompt 是不可信边界的一部分。

也就是说，`pi` 并不是单纯追求“好用”，而是在扩展性和安全边界之间做了宿主级治理。

---

## 5. 和 cc fork path 的核心差异

对照 `cc/src/tools/AgentTool/forkSubagent.ts`，两边最核心的差异越来越清楚了：

## cc 的 fork 路径
- fork 是核心 runtime 内建路径
- 会保留 parent assistant message
- 会构造 byte-identical prefix 以追求 prompt cache sharing
- 会生成 synthetic fork child rules
- 更强调：从主 agent 里面高性能地分叉 worker

一句话：

> cc 的 fork 是“核心产品内部的优化型分叉机制”。

## pi 的 subagent 路径
- subagent 是 extension 里的委派器
- 直接起新 `pi` 进程
- 默认不共享 session
- 强调 agent definition / tool/model/prompt 的宿主可配置性
- 更强调：让用户自己长出 orchestrator

一句话：

> pi 的 subagent 是“宿主之上的可编排委派机制”。

这也进一步验证前面的总判断：

- cc 更像强成品 agent
- pi 更像可编程宿主

---

## 当前最值得写进正文的判断

如果要写成文章，这一轮最值得收的一句是：

> `pi` 的 subagent 不是在主会话里假装多角色协作，而是把“多 agent 委派”明确做成 extension 层的独立 runtime 调度。`

而它最值得学的，不只是“会并发”，而是这 4 个结构点：

1. **独立进程隔离**
2. **agent-specific prompt / tool / model 注入**
3. **结构化事件流回传**
4. **project-agent 信任边界治理**

---

## 后续继续读什么

下一轮如果继续沿这条线推进，最值得补的是：

1. `renderResult()` 这一层的 UI 设计，看看宿主怎样把 subagent 结果重新做成主会话可读对象
2. `plan-mode` 示例，对比它和 subagent 都在 extension 层，分别代表哪两种工作流外化路线
3. `custom-provider-*`，验证 provider 也确实是宿主层可长能力，而不是 README 口号
