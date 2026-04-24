# 05｜当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么

![pi extension surface host platform](../assets/pi-ch05-extension-surface-host-platform.svg)

前面两章我们看了两个很具体的例子。

第 03 章，subagent 不是在主上下文里做 prompt 分身，而是由 extension 层启动独立 `pi` runtime，把任务委派出去。

第 04 章，plan mode 也不是一个内建 planner agent，而是同一个主 runtime 在只读探索态和执行态之间切换。

这两个例子看起来不同：一个是委派，一个是切模式。

但它们背后其实是同一件事：

> `pi` 把高级工作流从 core product 里外化出来，让它们长在 extension runtime surface 上。

所以第 05 章要把问题往上收一层：如果 tool、UI、provider、compaction、session lifecycle 都能外化，agent 产品会变成什么？

我的判断是：

> 它会从“固定功能集合”变成“可编程宿主平台”。

这也是 `pi` 和强成品 agent 路线最关键的差异。

---

## 1. extension 不是插件点，而是 runtime interface

很多产品说自己支持插件，实际含义可能只是：你可以多加几个 tool。

`pi` 的 extension surface 不止如此。

看 `packages/coding-agent/src/core/extensions/types.ts`，extension 能接触到的层非常多。

它可以注册 LLM 可调用工具：

```ts
registerTool(...)
```

可以注册命令、快捷键和 CLI flag：

```ts
registerCommand(...)
registerShortcut(...)
registerFlag(...)
```

可以改 UI：

```ts
ctx.ui.setStatus(...)
ctx.ui.setWidget(...)
ctx.ui.setFooter(...)
ctx.ui.setHeader(...)
ctx.ui.setEditorComponent(...)
registerMessageRenderer(...)
```

可以写 session state：

```ts
appendEntry(customType, data)
setSessionName(name)
setLabel(entryId, label)
```

可以改工具面：

```ts
getActiveTools()
getAllTools()
setActiveTools(toolNames)
refreshTools()
```

也可以触碰模型和 provider：

```ts
setModel(model)
registerProvider(name, config)
unregisterProvider(name)
```

再加上 lifecycle hooks，比如：

- `session_start`
- `session_before_compact`
- `session_compact`
- `session_before_tree`
- `before_agent_start`
- `before_provider_request`
- `after_provider_response`
- `tool_call`
- `tool_result`
- `turn_end`

这已经不是“给 agent 加几个小功能”的插件系统了。

它更像一组 runtime interface。

也就是说，extension 可以参与：

- agent 能用什么工具；
- 用户看到什么 UI；
- 模型请求怎么发；
- session 状态怎么保存；
- compaction 怎么做；
- agent 每轮开始前看到什么上下文；
- provider 请求前后怎么被改写和观察。

这就是 `pi` 的核心卖点开始变清楚的地方。

它不是说“我内建了更多功能”，而是说：

> 我把 agent 的多个关键层都暴露成可编程 surface。

---

## 2. provider 外化：基础设施也可以由 extension 接管

最能说明“这不是普通插件”的一个点，是 provider。

普通插件系统通常只能加 tool，很少让你扩展模型 provider。因为 provider 牵涉到：

- base URL；
- API key；
- OAuth；
- model list；
- streaming protocol；
- cost / context window / max tokens；
- provider-specific request / response 结构。

但 `pi` 的 `registerProvider()` 明确把这些交给 extension。

`types.ts` 里对它的注释很直接：

```ts
/**
 * Register or override a model provider.
 *
 * If `models` is provided: replaces all existing models for this provider.
 * If only `baseUrl` is provided: overrides the URL for existing models.
 * If `oauth` is provided: registers OAuth provider for /login support.
 * If `streamSimple` is provided: registers a custom API stream handler.
 */
registerProvider(name: string, config: ProviderConfig): void;
```

`custom-provider-anthropic` 示例更具体。

它直接注册一个 `custom-anthropic` provider：

```ts
export default function (pi: ExtensionAPI) {
  pi.registerProvider("custom-anthropic", {
    baseUrl: "https://api.anthropic.com",
    apiKey: "CUSTOM_ANTHROPIC_API_KEY",
    api: "custom-anthropic-api",

    models: [
      {
        id: "claude-opus-4-5",
        name: "Claude Opus 4.5 (Custom)",
        reasoning: true,
        input: ["text", "image"],
        cost: { input: 5, output: 25, cacheRead: 0.5, cacheWrite: 6.25 },
        contextWindow: 200000,
        maxTokens: 64000,
      },
      // ...
    ],

    oauth: {
      name: "Custom Anthropic (Claude Pro/Max)",
      login: loginAnthropic,
      refreshToken: refreshAnthropicToken,
      getApiKey: (cred) => cred.access,
    },

    streamSimple: streamCustomAnthropic,
  });
}
```

这说明 provider 不只是配置项，而是 extension 可以接管的基础设施层。

这对系统型用户很重要。

因为很多真实环境里，模型访问并不是“填一个官方 API key”这么简单。你可能需要：

- 接公司内部模型网关；
- 接自建 proxy；
- 改 OAuth 登录；
- 按团队成本模型重写 price metadata；
- 支持一个还没进入官方 provider list 的 API；
- 对请求和响应做定制处理。

强成品 agent 通常会把 provider support 当产品能力：官方支持哪个，你就用哪个。

`pi` 的路线更像：provider 也可以长在宿主层。

这就是“宿主平台”和“功能产品”的差异。

---

## 3. compaction 外化：上下文治理也不是封闭 core 行为

另一个关键点是 compaction。

上下文压缩通常会被看成 agent core 的内部能力。因为它关系到：

- 哪些历史要保留；
- 哪些历史要总结；
- summary 怎么写；
- 从哪个 entry 开始保留；
- 压缩后模型看到什么。

如果这个能力完全封闭在 core 里，extension 顶多只能触发一下。

但 `pi` 暴露了 `session_before_compact`。

`types.ts` 里定义：

```ts
export interface SessionBeforeCompactEvent {
  type: "session_before_compact";
  preparation: CompactionPreparation;
  branchEntries: SessionEntry[];
  customInstructions?: string;
  signal: AbortSignal;
}
```

对应的 result 允许 extension：

```ts
export interface SessionBeforeCompactResult {
  cancel?: boolean;
  compaction?: CompactionResult;
}
```

也就是说，extension 不只是收到通知。它可以取消，也可以直接返回自定义 compaction 结果。

`AgentSession.compact()` 里的流程也证明了这一点：

```ts
if (this._extensionRunner.hasHandlers("session_before_compact")) {
  const result = await this._extensionRunner.emit({
    type: "session_before_compact",
    preparation,
    branchEntries: pathEntries,
    customInstructions,
    signal: this._compactionAbortController.signal,
  });

  if (result?.cancel) {
    throw new Error("Compaction cancelled");
  }

  if (result?.compaction) {
    extensionCompaction = result.compaction;
    fromExtension = true;
  }
}
```

如果 extension 给了 compaction，就用 extension 的结果；否则才走默认 compact。

最后它还会把 compaction 写回 session，并发出 `session_compact` 事件：

```ts
this.sessionManager.appendCompaction(summary, firstKeptEntryId, tokensBefore, details, fromExtension);

await this._extensionRunner.emit({
  type: "session_compact",
  compactionEntry: savedCompactionEntry,
  fromExtension,
});
```

这说明 compaction 也被做成了 host behavior。

这件事比“支持自定义摘要 prompt”更深。

它意味着 extension 可以参与上下文治理，而上下文治理正是长期 agent 工作的核心能力之一。

如果说 session tree 是 runtime state 的存储底座，那么 compaction hook 就是 extension 参与 runtime state 重写的入口。

---

## 4. session custom entry 是 workflow state 的落点

前面几章反复提到一个东西：custom entry。

在 extension API 里，它叫：

```ts
appendEntry<T = unknown>(customType: string, data?: T): void;
```

落到 `AgentSession` 里，是：

```ts
appendEntry: (customType, data) => {
  this.sessionManager.appendCustomEntry(customType, data);
}
```

这个接口看起来很小，但它是宿主路线里非常关键的一块。

因为一旦 extension 可以把自己的状态写进 session，它就不只是一个内存里的临时脚本。

它可以把 workflow state 变成 session tree 的一部分。

第 04 章的 plan mode 已经展示了这一点：

- plan mode 把 `enabled`、`todos`、`executing` 写进 custom entry；
- session resume 时再从 entries 里恢复；
- 执行态还会扫描 `[DONE:n]`，重建 todo completion。

这意味着 session 里保存的不是“聊天记录”。

它保存的是：

- 用户和模型的消息；
- 工具调用结果；
- compaction；
- branch summary；
- labels；
- extension 自己的 workflow state。

这正是第 02 章 session tree 的后续含义。

当 session 能保存 extension state，agent 产品就不再只是一个对话窗口，而开始变成一个可以恢复、分叉、压缩、扩展的工作台。

---

## 5. 从功能集合到宿主平台

现在可以把这条线收起来了。

如果一个 agent 产品的能力主要来自 core product，那么它的竞争方式会很像功能表竞争：

- 谁内建的 worker 更多；
- 谁内建的 plan 更好；
- 谁内建的 provider 更多；
- 谁内建的 UI 更完整；
- 谁内建的上下文策略更聪明。

这是一条强成品路线。

Claude Code / cc 很大程度上就是这条路线：官方把很多关键工作流做进产品内部，用户直接用。

`pi` 不是完全相反。它也有 core，也有默认工具，也有默认 session 和默认 provider。

但它更有意思的地方在于：

> 它把很多原本会被封进 core 的东西，改成了 extension 可以参与的 runtime surface。

于是 agent 产品的形态开始变化。

它不只是“一个功能集合”，而是更像“一个宿主平台”：

- tool 可以外化；
- UI 可以外化；
- command / shortcut / flag 可以外化；
- provider 可以外化；
- compaction 可以外化；
- session state 可以外化；
- 高级工作流可以外化。

这就是为什么我一直说，`pi` 的卖点不是“比 cc 多了什么功能”。

更准确地说：

> `pi` 的卖点，是它允许系统型用户把自己的 agent 工作流长到宿主里。

---

## 6. 这条路线对谁值钱

这条路线不一定适合所有人。

如果一个用户只想要开箱即用、官方替你决定好所有流程，那强成品 agent 可能更舒服。

但对另一类用户，`pi` 的价值会很明显。

比如：

- 你想把公司内部模型网关接进来；
- 你想让 agent 使用团队自己的审批和权限策略；
- 你想做 repo-local 的工作流；
- 你想自定义 compaction，让摘要符合团队上下文；
- 你想把 plan、review、test、commit 做成自己的流程；
- 你想控制 UI，而不是接受官方固定展示；
- 你想在 session tree 上保存自己的状态。

这类用户关心的不是“官方菜单里有没有这个功能”。

他们关心的是：

> 我能不能把自己的系统接进去？我能不能把自己的 workflow 长出来？

对这类人，`pi` 的 host/runtime 路线很有吸引力。

---

## 7. 代价：越像宿主，越需要治理

但这条路线不是免费午餐。

越开放，治理复杂度越高。

如果 extension 可以注册 tool，就要考虑 tool 权限。

如果 extension 可以改 UI，就要考虑 UI 行为和用户信任。

如果 extension 可以注册 provider，就要考虑凭据、OAuth、请求改写、数据出境。

如果 extension 可以改 compaction，就要考虑摘要质量、上下文丢失、审计难度。

如果 extension 可以写 custom entry，就要考虑 session schema、恢复逻辑、分叉语义。

如果 project-local agent 或 project-local extension 被加载，就要考虑 repo-controlled prompt / code 是否可信。

第 03 章 subagent 已经碰到这个问题：project agents 默认不加载，启用 project/both 时还要确认。

第 04 章 plan mode 也碰到这个问题：只读不是靠 prompt，而是要用 runtime enforcement 做 bash allowlist。

所以 `pi` 的宿主路线有一个天然代价：

> 它把更多能力交给用户和 extension，也就把更多治理责任推到了宿主边界上。

这不是缺点，也不是优点，而是这条产品路线的价格。

强成品 agent 把更多决定收在官方 core 里，用户少操心，但可塑性低。

可编程宿主把更多 surface 放出来，用户能长出自己的系统，但需要更强的工程能力和治理意识。

---

## 结语：pi 的价值不是功能最多，而是可继续生长

现在回到本章开头的问题：当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么？

答案是：它会更像一个宿主平台。

`pi` 的关键价值，不是它现在内建了多少功能，而是它把 agent 的关键层拆成了可以继续编程的 surface。

这也让前几章串起来了：

- session tree：让工作过程变成 runtime state；
- subagent：让任务委派长在 extension 层；
- plan mode：让主 runtime 切出新的工作模式；
- provider：让模型基础设施也能被 extension 接管；
- compaction：让上下文治理也能被 extension 介入；
- custom entry：让 extension state 能写回 session。

所以 `pi` 代表的不是“另一个 Claude Code”。

它更像另一种 agent 产品路线：

> 少一点官方强成品，多一点宿主 surface；少一点固定流程，多一点可生长工作台。

这条路线不会让所有人都更轻松。

但对会自己搭 agent 工作台的人，它很值得研究。
