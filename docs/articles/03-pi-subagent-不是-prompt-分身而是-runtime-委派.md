# 03｜pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派

![pi subagent runtime delegation](../assets/pi-ch03-subagent-runtime-delegation.svg)

看到 subagent 这个词，很多人第一反应是：这不就是多写几个角色 prompt 吗？

比如一个叫 planner，一个叫 reviewer，一个叫 worker。主 agent 仍然待在同一个上下文里，只是让模型临时换一种口吻、换一种职责，假装自己是另一个 agent。

这种做法当然也有用，但它不是 `pi` 这套 subagent 示例最值得看的地方。

`pi` 的 subagent 更像另一件事：

> 它不是在主上下文里模拟多个角色，而是由 extension 层启动独立 `pi` 进程，把任务委派给另一个 agent runtime，再把结构化结果带回主会话。

这就是第 03 章要讲的核心。

第 01 章我们说，`pi` 不像一个强成品 agent，更像一个可编程宿主。第 02 章我们看到，它连 session 都不是按线性聊天记录来做，而是保存成可以分叉、回放、压缩的 runtime state。

到了 subagent 这一章，这个判断又往前走了一步：`pi` 不只是保存 runtime，它还允许高级工作流从 extension 层长出来。

---

## 1. subagent 首先不是 core 功能，而是 extension 注册出来的 tool

先看入口。

`packages/coding-agent/examples/extensions/subagent/index.ts` 里，subagent 是通过 extension API 注册出来的：

```ts
export default function (pi: ExtensionAPI) {
  pi.registerTool({
    name: "subagent",
    label: "Subagent",
    description: [
      "Delegate tasks to specialized subagents with isolated context.",
      "Modes: single (agent + task), parallel (tasks array), chain (sequential with {previous} placeholder).",
      'Default agent scope is "user" (from ~/.pi/agent/agents).',
      'To enable project-local agents in .pi/agents, set agentScope: "both" (or "project").',
    ].join(" "),
    parameters: SubagentParams,
    async execute(...) {
      // ...
    },
  });
}
```

这个入口很重要。

如果只从功能上看，你会说：`pi` 支持 subagent。可是从产品路线看，更关键的是：`subagent` 并不是写死在 core product 里的一条内建工作流，而是一个 extension tool。

也就是说，`pi` 没有把高级工作流完全收进官方内核，而是提供了足够强的 extension surface，让这种工作流可以在外层长出来。

这和 Claude Code / cc 的感觉不一样。cc 里的 subagent/fork 更像 core runtime 的内部机制：它要追求主 agent 内部分叉、共享 prefix、复用 prompt cache、控制 fork child rules。那是一条很强的官方产品路径。

`pi` 这里的重点不是“官方替你做一个最优 subagent”，而是“宿主允许你在 extension 层实现一个 subagent 调度器”。

这正是全书反复说的差异：

> cc 更像强成品 agent；`pi` 更像可编程宿主。

---

## 2. 真正的隔离来自独立 `pi` 进程

那这个 extension 到底怎么委派任务？

关键在 `runSingleAgent()`。

它不是把一个新的 prompt 塞回主会话，而是组装一组 CLI 参数，然后启动一个新的 `pi` 进程：

```ts
const args: string[] = ["--mode", "json", "-p", "--no-session"];
if (agent.model) args.push("--model", agent.model);
if (agent.tools && agent.tools.length > 0) args.push("--tools", agent.tools.join(","));

if (agent.systemPrompt.trim()) {
  const tmp = await writePromptToTempFile(agent.name, agent.systemPrompt);
  args.push("--append-system-prompt", tmp.filePath);
}

args.push(`Task: ${task}`);

const invocation = getPiInvocation(args);
const proc = spawn(invocation.command, invocation.args, {
  cwd: cwd ?? defaultCwd,
  shell: false,
  stdio: ["ignore", "pipe", "pipe"],
});
```

这里有几个细节不能跳过。

第一个是 `--no-session`。

它说明子 agent 默认不是共享主会话历史，而是一次独立运行。主会话把任务交出去，子 agent 在自己的上下文窗口里完成。

第二个是 `--append-system-prompt`。

agent definition 不是一个展示标签，而是真的可以把自己的 system prompt 写入临时文件，再附加到子进程里。

第三个是 `--tools` 和 `--model`。

这意味着不同 agent 不只是“性格不同”，还可以有不同工具集合和模型配置。

所以 `pi` 的 subagent 至少包含这几层隔离：

- 独立进程；
- 独立上下文；
- agent-specific system prompt；
- agent-specific tools；
- agent-specific model。

这就不是 prompt 分身了。

更准确地说，它是：

> 主 runtime 通过 extension 启动另一个 agent runtime。

---

## 3. 父进程消费的不是文本，而是结构化事件流

如果只是起一个子进程，然后读一段 stdout，那还只是 shell out。

`pi` 这套实现更进一步：它让子进程以 JSON mode 运行，然后父进程逐行解析事件。

源码里有这样一段：

```ts
const processLine = (line: string) => {
  if (!line.trim()) return;
  let event: any;
  try {
    event = JSON.parse(line);
  } catch {
    return;
  }

  if (event.type === "message_end" && event.message) {
    const msg = event.message as Message;
    currentResult.messages.push(msg);
    // accumulate usage, turns, model, stopReason...
    emitUpdate();
  }

  if (event.type === "tool_result_end" && event.message) {
    currentResult.messages.push(event.message as Message);
    emitUpdate();
  }
};
```

这段的含义是：父进程拿到的不是“子 agent 最后说了一句话”，而是另一个 agent runtime 的结构化事件流。

所以主会话可以知道：

- 子 agent 什么时候产出 assistant message；
- 子 agent 什么时候完成 tool result；
- 子 agent 用了多少 input/output token；
- cache read/write 是多少；
- cost 是多少；
- context tokens 是多少；
- stop reason 是什么；
- 用的是哪个 model。

这件事很关键。

因为一旦结果是结构化事件流，宿主就可以重新渲染、折叠、展开、汇总 usage，而不是只能贴一段最终文本。

这也是为什么我说它更像 runtime 委派，而不是 prompt 角色扮演。

角色扮演的边界是“模型怎么说”；runtime 委派的边界是“另一个 agent 怎么运行、怎么回传、怎么被宿主重新接管”。

---

## 4. single / chain / parallel 是最小 orchestration 骨架

`pi` 的 subagent 示例也不只支持一个 worker。

它定义了三种模式：

```ts
// Single: { agent: "name", task: "..." }
// Parallel: { tasks: [{ agent: "name", task: "..." }, ...] }
// Chain: { chain: [{ agent: "name", task: "... {previous} ..." }, ...] }
```

再看执行逻辑：

- single：调用一次 `runSingleAgent()`；
- chain：按顺序执行，每一步可以把 `{previous}` 替换成上一步输出；
- parallel：用并发限制跑多个任务，`MAX_PARALLEL_TASKS = 8`，`MAX_CONCURRENCY = 4`。

这说明它不是一个只展示概念的 demo。

它已经具备一个多 agent orchestration 的最小骨架：

1. 有 agent discovery；
2. 有任务分发；
3. 有串行依赖；
4. 有并行执行；
5. 有 streaming update；
6. 有 usage 汇总；
7. 有错误中断。

当然，这不等于 `pi` 已经是成熟的多 agent 平台。更准确的说法是：

> `pi` 展示了一个高级工作流怎样从 extension 层长出来。

这比“它支持 subagent”更重要。

因为如果宿主 surface 足够稳定，今天可以长出 subagent，明天也可以长出 plan mode、review workflow、research workflow、CI assistant、文档 agent、repo-local worker。

`pi` 这条路线最值得看的地方，不是某一个内建功能有多完整，而是这些工作流不必都进入 core 才能运行。

---

## 5. project agents 暴露了宿主路线的信任边界

还有一个很容易被忽略的点：agent 从哪里来。

`agents.ts` 支持两类 agent definition：

```ts
export type AgentScope = "user" | "project" | "both";
```

发现逻辑大概是：

- user agents 来自 `~/.pi/agent/agents/*.md`；
- project agents 来自 `.pi/agents/*.md`；
- 默认只加载 user-level agents；
- 如果 scope 是 `project` 或 `both`，才会加载项目内 agents。

入口处还特别写了说明：

```ts
'Default agent scope is "user" (from ~/.pi/agent/agents).',
'To enable project-local agents in .pi/agents, set agentScope: "both" (or "project").',
```

并且在交互环境里，如果真的要运行 project-local agents，还会提示确认：

```ts
const ok = await ctx.ui.confirm(
  "Run project-local agents?",
  `Agents: ${names}\nSource: ${dir}\n\nProject agents are repo-controlled. Only continue for trusted repositories.`,
);
```

这不是小细节。

只要一个 repo 能放 `.pi/agents/*.md`，它就能提供 repo-local prompt。这个 prompt 可能让 agent 读文件、跑命令、修改代码。也就是说，project agent 天然属于信任边界的一部分。

`pi` 默认不加载 project agents，要求显式 scope，并在交互模式下提醒确认，说明作者知道这条宿主路线的代价。

越可编程，就越需要治理。

这也为后面的收口章埋了一个点：`pi` 的 host/runtime 路线确实更自由，但它不是免费午餐。extension、project agents、custom tools、custom providers 都会把产品从“固定功能集合”推向“可治理的运行平台”。

能力边界扩大，安全边界也必须跟着扩大。

---

## 6. 和 cc fork path 的差异

现在可以把差异收清楚了。

cc 的 fork/subagent 路线，更像核心产品内部的一条优化型分叉机制。

它关心的是：

- 如何从主 agent 高性能分出 worker；
- 如何保留 parent assistant message；
- 如何构造 byte-identical prefix；
- 如何最大化 prompt cache sharing；
- 如何通过 synthetic rules 管住 fork child。

这是一条很强的产品内建路线。

`pi` 的 subagent 路线则不一样。

它关心的是：

- extension 如何注册一个 subagent tool；
- 如何发现 user / project agents；
- 如何为每个 agent 注入 prompt / tools / model；
- 如何 spawn 独立 `pi` runtime；
- 如何用 JSON event stream 把结果带回主会话；
- 如何支持 single / chain / parallel；
- 如何处理 project-local agent 的信任边界。

所以两边不是“谁有没有 subagent”的区别，而是产品哲学不同。

一句话概括：

> cc 的 fork 是核心产品内部的优化型分叉机制；`pi` 的 subagent 是宿主之上的可编排 runtime 委派。

前者更像强成品 agent 的内部能力，后者更像可编程宿主暴露出来的生长面。

---

## 结语：subagent 证明高级工作流可以从宿主层长出来

第 02 章讲 session tree 时，我们看到 `pi` 没有把工作历史当成一条聊天记录，而是保存成 runtime state。

第 03 章看 subagent，则能看到另一层：`pi` 也没有把所有高级工作流都写死在 core product 里。

它让 extension 注册 tool，让 extension 发现 agents，让 extension spawn 新 `pi`，让 extension 消费 JSON event stream，再把结果渲染回主会话。

这就是 `pi` 最值得研究的地方。

它不是说：官方已经给你做好了一个最强 subagent。

它更像是在说：

> 只要宿主 runtime 足够开放，多 agent orchestration 就可以作为外部能力长出来。

这也是为什么 `pi` 不能只按功能表去读。

功能表会告诉你：它有 subagent。

源码会告诉你更重要的事：它把 subagent 放在 extension 层实现，说明它真正想做的不是堆功能，而是让 agent 工作流变成宿主可以继续生长的东西。
