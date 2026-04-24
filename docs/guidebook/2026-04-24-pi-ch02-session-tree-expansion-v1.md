# 第 02 章展开稿：为什么 pi 要把 session 做成树，而不是聊天记录

## 章节任务

第 01 章已经给出了总判断：`pi` 不是另一种“强成品 agent”，而是一个更愿意把工作流能力开放给宿主和扩展系统的 agent runtime。

第 02 章要把这个判断往下落一层：看它最底层怎样保存和恢复一次工作。

这一章不讲 `/tree`、`/fork`、`/clone` 怎么用，而是回答一个更底层的问题：

> 如果一个 agent 只是聊天机器人，session 做成一条线就够了。为什么 `pi` 要把 session 做成一棵树？

## 一句话判断

`pi` 的 session tree 不是为了让聊天记录看起来更高级，而是为了让一次 agent 工作可以被分叉、回放、压缩、标记、注入扩展状态，并在不同运行时之间安全迁移。

也就是说，session 在 `pi` 里不是 transcript，而是 runtime state log。

## 常见误读

### 误读一：树形 session 只是 UI 导航功能

表面看，用户能在 TUI 里看树、回到某个节点、从某条消息前重新开始。这容易让人以为 session tree 是一种“更好的聊天记录浏览器”。

源码里不是这样。树不是渲染层后补出来的，而是每一个 `SessionEntry` 的基础结构：

```ts
export interface SessionEntryBase {
  type: string;
  id: string;
  parentId: string | null;
  timestamp: string;
}
```

也就是说，`parentId` 从一开始就是 session entry 的基本字段，不是 UI 计算出来的派生关系。

### 误读二：fork 是复制一份历史

`fork` 的用户体验像“从这里重新开始”，但实现上更准确的说法是：

- 当前 session 内分支：移动 leaf pointer，让下一条 entry 接到旧节点下面；
- 新 session 文件分支：提取 root 到目标 leaf 的 active path，写成一个新的 session JSONL；
- 旧 entry 不修改、不删除。

这比“复制一份聊天记录然后继续写”更像版本控制。

### 误读三：compaction 只是上下文压缩

`pi` 的 compaction 不是把旧消息文本替换掉，而是在树上追加一个 `compaction` entry。`buildSessionContext()` 在构造模型上下文时识别它：先放 summary，再从 `firstKeptEntryId` 开始保留一段消息，然后接上 compaction 之后的消息。

所以压缩不是破坏历史，而是改变“当前 active branch 被编译给模型时”的上下文形态。

## 关键源码证据

### 1. session entry 天然带 parentId

文件：`packages/coding-agent/src/core/session-manager.ts`

核心结构：

```ts
export interface SessionEntryBase {
  type: string;
  id: string;
  parentId: string | null;
  timestamp: string;
}
```

所有 entry 都挂在这个基类之上：message、model change、thinking level change、compaction、branch_summary、custom、custom_message、label、session_info。

这说明 `pi` 记录的不是“用户消息 / 助手消息列表”，而是“多种状态事件组成的树”。

### 2. append 永远把新 entry 接到当前 leaf 后面

文件：`packages/coding-agent/src/core/session-manager.ts`

典型逻辑：

```ts
appendMessage(message) {
  const entry = {
    type: "message",
    id: generateId(this.byId),
    parentId: this.leafId,
    timestamp: new Date().toISOString(),
    message,
  };
  this._appendEntry(entry);
  return entry.id;
}
```

`_appendEntry()` 会做三件事：

1. 把 entry 追加到 `fileEntries`；
2. 放入 `byId` 索引；
3. 把 `leafId` 推进到新 entry。

这是一套 append-only 的工作日志模型。

### 3. buildSessionContext 从 leaf 回溯 active branch

文件：`packages/coding-agent/src/core/session-manager.ts`

关键逻辑：

```ts
let current = leaf;
while (current) {
  path.unshift(current);
  current = current.parentId ? byId.get(current.parentId) : undefined;
}
```

也就是说，模型并不是看到整棵树，而是看到当前 leaf 往回追溯出来的一条 active path。

这解释了一个关键差异：

- 存储层保存整棵树；
- 运行层选择一条分支；
- 上下文层把这条分支编译成 messages。

### 4. branch 不删除历史，只移动 leaf

文件：`packages/coding-agent/src/core/session-manager.ts`

```ts
branch(branchFromId: string): void {
  if (!this.byId.has(branchFromId)) {
    throw new Error(`Entry ${branchFromId} not found`);
  }
  this.leafId = branchFromId;
}
```

注释写得很直白：下一次 `appendXXX()` 会在指定 entry 后创建子节点，形成新分支；已有 entry 不会被修改或删除。

### 5. createBranchedSession 提取单条 path，而不是复制全树

文件：`packages/coding-agent/src/core/session-manager.ts`

```ts
const path = this.getBranch(leafId);
const pathWithoutLabels = path.filter((e) => e.type !== "label");
...
this.fileEntries = [header, ...pathWithoutLabels, ...labelEntries];
```

这让 `/fork` 可以产生一个新的 session 文件，但新文件只带走 active branch，不带走所有旁支。

### 6. runtime replacement 让 session 切换成为运行时事件

文件：`packages/coding-agent/src/core/agent-session-runtime.ts`

`switchSession()`、`newSession()`、`fork()` 都不是单纯换一个文件路径，而是：

1. 触发 before switch / before fork hook；
2. teardown 当前 session；
3. createRuntime；
4. apply 新 session 和 cwd-bound services；
5. finish replacement，重新绑定 session。

这说明 session tree 连接的不只是历史记录，还包括扩展系统、工具服务、cwd、session lifecycle。

## 论证骨架

### 第一层：线性聊天记录能做什么

线性聊天记录适合表达：

- 第 1 轮用户说了什么；
- 第 1 轮助手答了什么；
- 第 2 轮继续发生了什么。

它默认世界只有一条时间线。

### 第二层：agent 工作天然不是一条线

真实的 agent 工作经常出现：

- 试了一个方案，想回到中间节点换路线；
- 某条分支失败，但其中一段分析仍然有价值；
- 旧上下文太长，需要压缩，但不能丢掉可恢复性；
- 扩展需要把自己的状态写进 session，重启后恢复；
- 想把某个中间结果单独 fork 成一个新任务。

如果只有线性 transcript，这些需求都要靠“复制文本 + 人脑记忆 + 命名文件”来补。

### 第三层：pi 把这些都下沉到 session runtime

`pi` 的做法是把每个运行时事件都 entry 化，并且让 entry 具有 parentId：

- 普通消息是 entry；
- 模型切换是 entry；
- thinking level 变化是 entry；
- compaction 是 entry；
- branch summary 是 entry；
- extension custom state 是 entry；
- extension injected message 也是 entry。

这就让“重新从某处开始”不是 prompt 技巧，而是 runtime 操作。

## 和 Claude Code 的对比角度

这一章不要写成 “pi 比 Claude Code 更强”。更稳的说法是：

- Claude Code 把很多高级工作流做成产品内建体验；
- `pi` 把工作流的可变部分下沉成 session/runtime/extension 能力；
- 所以 `pi` 的 session tree 更像一个宿主能力接口，而不是单纯的聊天记录管理。

换句话说：

> Claude Code 给你一个设计好的驾驶舱；`pi` 先给你一套可分叉、可恢复、可注入状态的底盘。

## 正文结构建议

1. 开头：从一个具体场景切入——agent 做到一半发现路线错了，要回到中间节点重跑。
2. 解释线性聊天记录为什么不够。
3. 展示 `SessionEntryBase.parentId`：树不是 UI，而是数据模型。
4. 展示 append/leaf/path：session 是 append-only tree，模型只看 active branch。
5. 展示 fork/new/switch/runtime replacement：session 切换是运行时事件。
6. 收束到本书主线：这正是 `pi` 作为可编程宿主的底座。

## 配图建议

已补图：`docs/assets/pi-ch02-session-tree-runtime.svg`

图的重点不是画 API，而是画三层关系：

- 持久层保存整棵 entry tree；
- active leaf 选择一条 branch；
- `buildSessionContext()` 把 branch 编译成 messages 给 LLM。
