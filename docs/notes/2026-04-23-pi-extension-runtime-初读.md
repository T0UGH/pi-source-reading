# 2026-04-23 pi extension runtime 初读

## 结论

从第一轮源码看，`pi` 的 extension 不是简单插件，而是一整套 runtime surface。

这说明 `pi` 想做的不是“支持一些插件的 coding agent”，而是：

> **把 coding agent 本体做成宿主。**

---

## 关键锚点

- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/core/extensions/loader.ts`
- `packages/coding-agent/src/core/extensions/runner.ts`
- `packages/coding-agent/src/core/extensions/wrapper.ts`
- `packages/coding-agent/examples/extensions/README.md`

---

## 第一轮看到的能力边界

### 1. 不只是注册 tool
`types.ts` 里 extension 能做的事远超“加几个工具”：

- register tool
- register command / shortcut / flag
- 改 UI：footer / header / widget / overlay / editor
- 改 input / autocomplete
- 参与 session lifecycle
- 参与 compaction
- 注册 provider
- 调整 working indicator / hidden thinking / editor text

这意味着：

> extension 不是功能补丁，而是 runtime 接口。

### 2. loader 是认真做的平台兼容层
`loader.ts` 不是简单 `import()` 一下扩展，而是专门处理：
- TypeScript module loading
- jiti
- alias / virtual modules
- Bun binary compatibility

这说明 extension 不是开发态玩具，而是正式分发模型的一部分。

### 3. runner 负责的不是“执行几个 hook”这么简单
`runner.ts` 里有：
- session switch / fork / compact / tree 的 before hooks
- shutdown event
- stale runtime invalidation
- command / shortcut / UI wiring

这说明 extension 已经和 session runtime 紧耦合。

---

## 当前判断

如果说 Claude Code 更像“官方已经定义好的强工作流产品”，那 `pi` 更像：

> **一台 agent 主机，官方只提供最小但足够强的底座，其他高级工作流由 extension 自己长。**

---

## 后续已完成情况

这一轮留下的验证点，已经在后续章节中收束：

- `ExtensionRunner` 和 `AgentSession` 的绑定方式，已进入第 05 章的 extension surface 论证；
- stale runtime / session replacement 的治理逻辑，已作为“宿主必须治理长期 runtime”的证据收进第 05、06 章；
- compaction hook 已用于说明 `pi` 不是只开放工具，而是把 session lifecycle 也外化给 extension。
