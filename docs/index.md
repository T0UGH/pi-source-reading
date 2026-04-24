# pi 源码导读

这不是一份 `pi` 的安装教程，也不是 coding agent 功能大全。

这是一套围绕一个判断展开的中文源码导读小书：

> `pi` 的卖点，不是它比 Claude Code 多了什么功能，而是它把 agent 做成了可编程宿主。

如果你想知道 `pi` 为什么值得单独研究，建议不要从“它有哪些命令”“它支持哪些 provider”开始，而是先看它怎样把 session、subagent、plan mode、extension surface 这些能力组织成一个可被二次编排的 agent 宿主。

---

## 正式阅读入口

按下面顺序读即可：

| 顺序 | 文章 | 这一章解决的问题 |
|---|---|---|
| 00 | [这不是另一个 Claude Code：为什么 pi 值得单独研究](articles/00-这不是另一个-claude-code.md) | 为什么这本书不做功能大全，而要按“强成品 agent vs 可编程宿主”来读 |
| 01 | [pi 不是强成品 agent，而是可编程宿主](articles/01-pi-不是强成品-agent-而是可编程宿主.md) | `pi` 和 Claude Code 的产品路线差异到底是什么 |
| 02 | [为什么 pi 要把 session 做成树，而不是聊天记录](articles/02-pi-session-tree-不是聊天记录而是-runtime.md) | 为什么 session tree 是宿主路线的 runtime 底座 |
| 03 | [pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派](articles/03-pi-subagent-不是-prompt-分身而是-runtime-委派.md) | 为什么 subagent 是 extension 层的 runtime 委派 |
| 04 | [plan mode 不是内建 planner，而是宿主切出来的一种工作模式](articles/04-pi-plan-mode-不是内建-planner-而是-runtime-mode-switch.md) | 为什么 plan mode 是主 runtime 的 mode switch |
| 05 | [当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么](articles/05-pi-extension-surface-让-agent-产品变成宿主平台.md) | 为什么 extension surface 会把 agent 产品推向宿主平台 |
| 06 | [为什么 pi 代表另一种 agent 产品路线](articles/06-pi-为什么代表另一种-agent-产品路线.md) | `pi` 适合谁、不适合谁，以及它和强成品 agent 的关系 |

也可以先进入 [正文索引](articles/INDEX.md)。

---

## 这本小书怎么读

这组文章的阅读路径不是“从 UI 到命令再到 provider”的功能展开，而是从产品路线往 runtime 结构里拆：

1. 先判断 `pi` 不是强成品 agent，而是 coding-agent harness；
2. 再看 session tree 为什么是宿主路线的状态底座；
3. 再看 subagent 和 plan mode 为什么没有被塞进 core，而是从 extension/runtime 层长出来；
4. 最后回到 extension surface，理解为什么 tool、UI、provider、compaction、session lifecycle 都能变成宿主的可编程边界。

读完后你得到的不是一份功能清单，而是一种判断 agent 产品的视角：

> 当高级工作流不再只能由官方内建，而是可以在宿主层被组织、替换和生长，coding agent 就不再只是一个工具，而开始接近一个工作台底座。

---

## 过程材料

正式阅读请优先看上面的 `00-06` 正文。下面这些是写作过程中的材料，适合需要追溯论证来源时再看：

- [规划与展开稿](guidebook/INDEX.md)
- [源码细读笔记](notes/INDEX.md)
- [配图索引](assets/INDEX.md)
