# 2026-04-23 pi plan mode 实现细读 v1

## 结论

`pi` 的 plan mode 不是一个内建 plan agent，而是一个 extension 里的双态 runtime。

它的核心不是“多了一个 plan 命令”，而是：

- 先进入 **只读探索态**
- 用受限工具池和 bash allowlist 逼模型先产出计划
- 再由用户选择是否进入 **执行态**
- 执行态通过 `[DONE:n]` 标记、custom entries 和 session resume rescan 维持计划进度

一句话说：

> `pi` 把 plan mode 做成了一种可切换的宿主行为，而不是一个核心内建 agent。`

---

## 关键锚点

- `packages/coding-agent/examples/extensions/plan-mode/README.md`
- `packages/coding-agent/examples/extensions/plan-mode/index.ts`
- `packages/coding-agent/examples/extensions/plan-mode/utils.ts`
- 对照：
  - `cc/src/tools/AgentTool/builtInAgents.ts`

---

## 这轮看到的关键实现点

## 1. plan mode 的第一本质是“切工具池”，不是换 agent

在 `index.ts` 里最关键的两组常量是：

- `PLAN_MODE_TOOLS = ["read", "bash", "grep", "find", "ls", "questionnaire"]`
- `NORMAL_MODE_TOOLS = ["read", "bash", "edit", "write"]`

开启 plan mode 后：
- `pi.setActiveTools(PLAN_MODE_TOOLS)`

关闭 / 进入执行态后：
- `pi.setActiveTools(NORMAL_MODE_TOOLS)`

这意味着它的核心不是“叫一个 plan 专家 agent 出来”，而是：

> **把同一个宿主 runtime 暂时切到只读探索模式。**

这点和 Claude Code 很不一样。

### 对照 cc
在 `cc/src/tools/AgentTool/builtInAgents.ts` 里，`PLAN_AGENT` 是 built-in agent 体系的一部分。

也就是说：
- cc 的思路更像：**plan 是官方内建 worker 角色**
- pi 的思路更像：**plan 是宿主切换到一种受限工作模式**

---

## 2. plan mode 的第二本质是“运行时约束”，不是 prompt 建议

`index.ts` 里不只是改工具池，还在 `tool_call` hook 上拦截 bash：

- 只有 `planModeEnabled`
- 且工具是 `bash`
- 然后走 `isSafeCommand(command)`

而 `utils.ts` 里：
- 有 destructive patterns 黑名单
- 有 safe patterns 白名单
- 最终 `isSafeCommand = !isDestructive && isSafe`

也就是说，plan mode 不是只在 system prompt 里说“请只读分析”，而是：

> **在运行时真正把 bash 约束成只读 allowlist。**

这个实现很宿主化，因为它强调：
- prompt 说了不算
- runtime enforcement 才算

---

## 3. 它通过 `before_agent_start` 给模型注入双态上下文

在 `before_agent_start` hook 里，它会根据状态注入两类隐藏上下文：

### A. `[PLAN MODE ACTIVE]`
内容里明确告诉模型：
- 当前是只读探索态
- 可用工具受限
- 不能修改文件
- 要在 `Plan:` 头下输出 numbered plan

### B. `[EXECUTING PLAN - Full tool access enabled]`
切到执行态后，又会注入：
- remaining steps
- 按顺序执行
- 每完成一步要打 `[DONE:n]`

这说明 plan mode 在 `pi` 这里不是单段 prompt，而是：

> **随着宿主状态切换而变化的 runtime context layer。**

---

## 4. 它不是“计划生成器”，而是“计划状态机”

`plan-mode` 里最容易被忽略的一点，是它其实已经做了一个小型状态机。

状态变量至少有：
- `planModeEnabled`
- `executionMode`
- `todoItems`

而状态推进链是：

1. `/plan` 或 flag 开启 plan mode
2. 只读态产出 `Plan:`
3. `agent_end` 时抽取 numbered todos
4. UI 询问：
   - Execute the plan
   - Stay in plan mode
   - Refine the plan
5. 如果执行：
   - 退出只读态
   - 开启 executionMode
   - 恢复正常工具池
   - 自动发送一条执行消息
6. 执行过程中根据 `[DONE:n]` 标记推进进度
7. 全部完成后：
   - 自动清空执行态
   - 恢复正常工具访问
   - 发一个 plan complete message

所以它不是一个“出个计划”的小工具，而是：

> **一个建立在 extension 之上的轻量 workflow state machine。**

---

## 5. 它把计划状态持久化进 session，而不是只放内存

`persistState()` 里会：
- `pi.appendEntry("plan-mode", {...})`

而在 `session_start` 时，又会从：
- `ctx.sessionManager.getEntries()`
- 找 `customType === "plan-mode"`
- 恢复：
  - enabled
  - todos
  - executing

甚至还会在 resume 时：
- 找最后一个 `plan-mode-execute`
- 只扫描它之后的 assistant messages
- 重新用 `[DONE:n]` 规则把 todo completion 补回去

这个点很重要，因为它说明：

> `pi` 的 extension 不是一次性脚本，而是和 session runtime 深度整合的状态层。`

也再次验证了 `pi` 的路线：
- 宿主底座负责 session tree / custom entries
- 高级工作流在 extension 层生长

---

## 6. 这条路线和 subagent 正好形成互补

前一轮的 subagent 更像：
- **把任务委派给独立 runtime**

这一轮的 plan mode 更像：
- **把主 runtime 切换成另一种工作模式**

也就是说，`pi` 在 extension 层已经展示了两种非常不同的高级工作流外化路线：

### 路线 A：委派型
- subagent
- 独立进程
- 独立上下文
- 主会话做 orchestration

### 路线 B：模态切换型
- plan mode
- 同一会话内切换工具池 / 提示层 / 状态机
- 主会话本身进入新的运行模式

这对整本书很重要，因为它说明：

> `pi` 的“宿主化”不是口号，它已经能承载至少两类不同的高级工作流设计。`

---

## 和 cc 的差异，当前最值得收的一句

如果把这轮判断压成一句话，就是：

> `cc` 把 plan 做成 built-in worker 角色；`pi` 把 plan 做成宿主可切换的运行模式。`

这两者的差异不只是功能实现，而是产品哲学：

- cc：高级工作流优先内建进产品
- pi：高级工作流优先外化成 host behavior

---

## 当前最值得写进正文的点

如果这轮要沉淀到正文，最有价值的不是“plan mode 有哪些按钮”，而是：

1. 它通过 `setActiveTools()` 把工作模式切成只读探索态 / 执行态
2. 它通过 runtime hook 做 bash allowlist，而不是只靠 prompt 约束
3. 它通过 `custom entries + session restore` 把计划状态做进 session runtime
4. 它和 subagent 一起说明：pi 已经有两种不同的工作流外化机制

---

## 后续已完成情况

这轮细读后，后续已经补齐：

1. `custom-provider-*` 与 extension surface 线索已收进第 05 章；
2. extension runner / agent session 的治理问题已收进第 05、06 章；
3. 主体小书正文已按 `00-06` 补齐，不再停留在“继续拆 examples”的阶段。
