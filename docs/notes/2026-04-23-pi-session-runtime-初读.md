# 2026-04-23 pi session runtime 初读

## 结论

`pi` 的 session 不是普通聊天记录，而是树形 runtime。

它底层不是简单 append messages，而是：
- JSONL 持久化
- 每个 entry 有 `id`
- 每个 entry 有 `parentId`
- 当前 branch 再被 resolve 成真正送给 LLM 的 context

一句话说：

> `pi` 把 session 当成树，而不是聊天历史。

---

## 关键锚点

- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

---

## 第一轮看到的关键实现点

### 1. `buildSessionContext()` 是真正的上下文重建器
关键逻辑：
- 从当前 `leafId` 找 leaf
- 从 leaf 一路沿 `parentId` 回溯到 root
- 得到 active path
- 再把 path 编译成 messages

这意味着：

> 文件里保存的是整棵树，模型每轮吃的是当前 branch 的 resolved context。

### 2. compaction 不是历史覆盖，而是树上的特殊 entry
`buildSessionContext()` 里专门处理 compaction entry：
- 先发 summary
- 再保留 firstKeptEntryId 之后的消息
- 再接 compaction 后的消息

这说明 compaction 也被整合进 session runtime，而不是单独外挂。

### 3. session replacement 是 runtime 级操作
`agent-session-runtime.ts` 里能看到：
- `switchSession()`
- `newSession()`
- `fork()`
- teardown current runtime
- create next runtime
- rebind session

这说明 session 切换不是 UI 操作，而是 runtime replacement。

---

## 为什么这点重要

因为如果 session 只是线性消息历史：
- `/tree` 会是很勉强的附属功能
- `/fork` 会更像复制聊天记录
- compaction / branch summary / custom messages 很难自然接入

而在 `pi` 这里，这些能力都建立在同一套树形 session 底座上。

---

## 当前判断

`pi` 之所以更像宿主，而不只是聊天壳，一个重要原因就是：

> 它把 session 做成了长期 runtime 的树形底座。

这件事对 branch、fork、compaction、replaced session context、extension hooks 都是基础。

---

## 后续已完成情况

这一轮留下的追问，已经分别进入后续章节：

- `AgentSession` 里 extension runner 的失效 / 重新绑定，已成为第 05 章讨论宿主治理面的材料；
- stale instance 机制，已用于说明宿主路线必须面对长期 runtime 的状态一致性问题；
- tree session 和 subagent / handoff 的关系，已在第 02、03 章中收束为“session tree 是 runtime 委派的底座”。
