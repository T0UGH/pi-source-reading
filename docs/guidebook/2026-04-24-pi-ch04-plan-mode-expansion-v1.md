# 2026-04-24 pi ch04 plan mode expansion v1

## 章节定位

第 04 章接在第 03 章 subagent 后面。

第 03 章证明的是：`pi` 可以在 extension 层长出“委派型”高级工作流——主 runtime 调度独立子 `pi` 进程。

第 04 章要证明另一种外化路线：`pi` 也可以在 extension 层把同一个主 runtime 切成另一种工作模式。

所以本章的核心不是“`pi` 有 plan mode”，而是：

> `pi` 的 plan mode 不是内建 planner 角色，而是 extension 驱动的 runtime mode switch。

它要服务全书主线：`pi` 的高级工作流并不都要写进 core product，而是可以通过宿主 surface 外化出来。

---

## 正文应该回答的问题

### 1. 为什么 plan mode 不是一个 planner agent？

源码里没有把 plan 做成独立 built-in worker。

`plan-mode/index.ts` 里最关键的是两组工具池：

- `PLAN_MODE_TOOLS = ["read", "bash", "grep", "find", "ls", "questionnaire"]`
- `NORMAL_MODE_TOOLS = ["read", "bash", "edit", "write"]`

进入 plan mode 时：

- `pi.setActiveTools(PLAN_MODE_TOOLS)`

退出 plan mode / 进入执行态时：

- `pi.setActiveTools(NORMAL_MODE_TOOLS)`

所以它不是另起一个 planner agent，而是同一个 runtime 改变自己的可用能力。

正文里可以这样讲：

> plan mode 的第一本质不是“换角色”，而是“切工具池”。

---

### 2. 为什么 runtime enforcement 比 prompt 更重要？

plan mode 不只是提示模型“不要改文件”。

它在 `tool_call` hook 上拦截 bash：

- 只有 plan mode enabled；
- 且工具名是 `bash`；
- 才检查 `isSafeCommand(command)`；
- 不安全就返回 `{ block: true, reason: ... }`。

`utils.ts` 里同时有：

- destructive patterns 黑名单；
- safe patterns 白名单；
- `isSafeCommand = !isDestructive && isSafe`。

这说明：

> `pi` 的 plan mode 不是 prompt 建议，而是宿主层的运行时约束。

这是本章最重要的源码证据之一。

---

### 3. before_agent_start 为什么说明它是 context layer？

在 `before_agent_start` hook 里，extension 会根据当前状态注入不同隐藏上下文。

如果是只读探索态，注入：

- `[PLAN MODE ACTIVE]`
- 只读工具限制；
- 不能 edit/write；
- bash 受 allowlist 限制；
- 要输出 `Plan:` header 下的 numbered plan。

如果是执行态，注入：

- `[EXECUTING PLAN - Full tool access enabled]`
- remaining steps；
- 按顺序执行；
- 每完成一步输出 `[DONE:n]`。

这意味着 plan mode 的提示不是一次性 prompt，而是随着宿主状态变化的 runtime context layer。

---

### 4. 为什么它是状态机？

它内部至少有三个状态变量：

- `planModeEnabled`
- `executionMode`
- `todoItems`

状态推进链：

1. `/plan` 或 `--plan` 开启只读探索态；
2. 工具池切到 `PLAN_MODE_TOOLS`；
3. agent 输出 `Plan:`；
4. `agent_end` 抽取 numbered todos；
5. UI 让用户选择 Execute / Stay / Refine；
6. Execute 后切到执行态；
7. 工具池恢复 `NORMAL_MODE_TOOLS`；
8. 自动发送执行消息；
9. 执行中用 `[DONE:n]` 标记完成；
10. 全部完成后清空状态并发送 plan complete。

这不是“生成计划”这么简单，而是一个轻量 workflow state machine。

---

### 5. 为什么 session persistence 是宿主路线的关键证据？

`persistState()` 会：

- `pi.appendEntry("plan-mode", {...})`

`session_start` 时再从 session entries 中恢复：

- enabled
- todos
- executing

如果是 resume，它还会找到最后一个 `plan-mode-execute`，只扫描之后的 assistant messages，用 `[DONE:n]` 重新补全 todo completion。

这说明 extension 不是一次性脚本，而是嵌入 session runtime 的状态层。

它和第 02 章 session tree 能接上：

> session 不只是保存聊天记录，也保存 extension 的 workflow state。

---

## 建议正文结构

### 标题

04｜plan mode 不是内建 planner，而是宿主切出来的一种工作模式

### 开头

从误解切入：很多人看到 plan mode，会以为是多了一个官方 planner agent。

直接给判断：`pi` 的实现更像 runtime mode switch。

### 1. plan mode 的第一步是切工具池

讲 `PLAN_MODE_TOOLS` / `NORMAL_MODE_TOOLS` 和 `setActiveTools()`。

### 2. 只读不是靠自觉，而是靠 tool_call hook

讲 bash allowlist、destructive patterns、安全命令白名单。

### 3. before_agent_start 注入的是状态化 context

讲 `[PLAN MODE ACTIVE]` 和 `[EXECUTING PLAN...]` 两种上下文。

### 4. 它是一个计划状态机

讲 `Plan:` 提取、UI 选择、execute message、`[DONE:n]`、completion。

### 5. 它把 workflow state 写回 session

讲 `appendEntry("plan-mode")`、`session_start` 恢复、resume rescan。

### 6. 和 subagent 构成两种外化路线

收束：

- subagent：委派型，启动独立 runtime；
- plan mode：模态切换型，改变主 runtime 的工作方式。

---

## 建议配图

文件：`docs/assets/pi-ch04-plan-mode-runtime-switch.svg`

图要表达：

1. plan-mode extension 是一个轻量 workflow state machine；
2. 只读探索态切 `PLAN_MODE_TOOLS`，并用 bash allowlist enforce；
3. 计划抽取后由用户选择 Execute / Stay / Refine；
4. 执行态恢复 `NORMAL_MODE_TOOLS`，用 `[DONE:n]` 推进；
5. custom entries + session_start restore 让状态跨 session 存活。

底部结论：

> plan 是主 runtime 的模式切换，不是另起一个官方 planner。

---

## 最该避免的写法

1. 不要写成 plan mode 使用教程。
2. 不要把重点放在 `/plan` 命令上。
3. 不要只写“工具变少了”，要写 runtime enforcement。
4. 不要遗漏 session persistence；这是它作为 host behavior 的关键证据。
5. 不要把它和 subagent 写成同一种能力。两者正好不同：一个是委派，一个是模态切换。
