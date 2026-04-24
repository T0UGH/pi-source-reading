# 2026-04-24 pi ch05 extension surface expansion v1

## 章节定位

第 05 章是前四章之后的收口章。

前面几章已经分别证明：

1. `pi` 不是强成品 agent，而是可编程宿主；
2. session tree 让工作过程变成 runtime state；
3. subagent 展示了“委派型”高级工作流外化；
4. plan mode 展示了“模态切换型”高级工作流外化。

第 05 章要把这些证据收成一个产品判断：

> 当 tool、UI、provider、compaction、session lifecycle 都能外化，agent 产品就不再只是固定功能集合，而会更像一个可编程宿主平台。

这章要同时讲价值和代价。

---

## 正文应该回答的问题

### 1. extension surface 到底覆盖了哪些层？

`types.ts` 说明 extension 能做的事情不只是注册 tool。

至少包括：

- custom tool：`registerTool()`；
- command / flag / shortcut：`registerCommand()`、`registerFlag()`、`registerShortcut()`；
- UI：`setStatus()`、`setWidget()`、`setFooter()`、`setHeader()`、`setEditorComponent()`、`registerMessageRenderer()`；
- session state：`appendEntry()`、`setSessionName()`、`setLabel()`；
- tool surface：`getActiveTools()`、`setActiveTools()`、`refreshTools()`；
- model/provider：`setModel()`、`registerProvider()`、`unregisterProvider()`；
- lifecycle hooks：`session_start`、`session_before_compact`、`session_compact`、`before_agent_start`、`before_provider_request` 等。

这说明 extension 是 runtime interface，不是普通插件点。

---

### 2. provider 为什么是关键证据？

`registerProvider()` 支持：

- 新 provider；
- override baseUrl；
- 自定义 models；
- OAuth；
- custom `streamSimple`。

`custom-provider-anthropic` 示例里，extension 直接注册 `custom-anthropic`，包含 custom API、models、OAuth login / refresh、streaming implementation。

这说明 provider 不再只是配置文件里的选项，而是 extension 可以接管的基础设施层。

---

### 3. compaction 为什么也是 host behavior？

`SessionBeforeCompactEvent` 允许 extension 在压缩前介入：

- 看到 `preparation`；
- 看到 `branchEntries`；
- 可以 cancel；
- 可以直接返回自定义 `compaction`。

`AgentSession.compact()` 会先 emit `session_before_compact`。如果 extension 返回 compaction，就用 extension 的结果；否则才走默认 compact。完成后还会 append compaction entry，并 emit `session_compact`。

这说明 compaction 也不是封闭 core 行为，而是可外化的 runtime step。

---

### 4. session state 为什么把这些能力串起来？

`appendEntry()` 会落到 `sessionManager.appendCustomEntry(customType, data)`。

plan mode 已经证明：extension 可以把 workflow state 写进 session，然后在 `session_start` 恢复。

这意味着 session tree 不只是保存聊天记录，而是保存 extension 与 runtime 共同演化的状态。

这就是 host/platform 的关键底座。

---

### 5. 这条路线对谁值钱？

它对普通“只想开箱即用”的用户未必最舒服。

但对系统型用户很值钱：

- 想自己做 workflow；
- 想换 provider；
- 想接公司内部模型网关；
- 想控制权限和安全边界；
- 想把团队流程做进 agent；
- 想把 agent 当工作台，而不是单一产品。

---

### 6. 代价是什么？

必须写代价，否则会像宣传稿。

代价至少包括：

- 治理复杂度上升；
- extension 权限边界需要设计；
- project-local agent / provider / tool 都有信任问题；
- UI 和状态持久化会增加心智负担；
- 用户需要工程能力；
- core 和 extension 的生命周期关系更难管理，比如 stale runtime / session replacement。

所以结论不是“这种路线一定赢”，而是：

> `pi` 用更高的治理复杂度，换来了让高级工作流在宿主层继续生长的能力。

---

## 建议正文结构

### 标题

05｜当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么

### 开头

承接 03/04：subagent 和 plan mode 不是两个孤立例子，而是 extension surface 的结果。

### 1. extension 不是插件点，而是 runtime interface

列出 `types.ts` 暴露的 surface。

### 2. provider 外化说明基础设施也能被 extension 接管

讲 `registerProvider()` 和 `custom-provider-anthropic`。

### 3. compaction 外化说明上下文治理也能被 extension 接管

讲 `session_before_compact` / `session_compact`。

### 4. session custom entry 是 workflow state 的落点

讲 `appendEntry()` 与 plan mode 的持久化。

### 5. agent 产品从功能集合变成宿主平台

收束产品判断。

### 6. 代价：治理、权限、生命周期、工程门槛

写清楚不是万能路线。

---

## 建议配图

文件：`docs/assets/pi-ch05-extension-surface-host-platform.svg`

图要表达：

1. 左侧是核心 agent runtime；
2. 中间是 extension runtime surface；
3. 右侧分两类结果：高级工作流、可替换基础设施；
4. 底部结论：价值不在功能最多，而在可继续长工作流。

---

## 最该避免的写法

1. 不要写成 extension API 列表。
2. 不要只夸开放性，要讲治理代价。
3. 不要把 provider/compaction 一笔带过，它们是“基础设施也能外化”的关键证据。
4. 不要离开前文主线：05 是对 01-04 的收束，不是新开一条插件教程。
