# pi-source-reading

> 一套围绕 **pi 为什么不是另一个 Claude Code，而是一种可编程 coding-agent 宿主** 展开的中文源码导读小书。

这个仓库不做 `pi` 的大而全说明书。

它只回答三个问题：

1. `pi` 和 Claude Code 最大的差异点是什么？
2. `pi` 真正的卖点是什么？
3. 为什么它对会自己搭 agent 工作台的人特别值得研究？

一句话说：

> `pi` 的卖点，不是它比 Claude Code 多了什么功能，而是它把 agent 做成了可编程宿主。

---

## 阅读入口

建议从 00 章开始顺序读：

| 顺序 | 文章 | 这一章回答什么 |
|---|---|---|
| 00 | [这不是另一个 Claude Code：为什么 pi 值得单独研究](docs/articles/00-这不是另一个-claude-code.md) | 为什么这本书不做功能大全，而要按“强成品 agent vs 可编程宿主”来读 |
| 01 | [pi 不是强成品 agent，而是可编程宿主](docs/articles/01-pi-不是强成品-agent-而是可编程宿主.md) | `pi` 和 Claude Code 的产品路线差异到底是什么 |
| 02 | [为什么 pi 要把 session 做成树，而不是聊天记录](docs/articles/02-pi-session-tree-不是聊天记录而是-runtime.md) | 为什么 session tree 是宿主路线的底座 |
| 03 | [pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派](docs/articles/03-pi-subagent-不是-prompt-分身而是-runtime-委派.md) | 为什么 subagent 是 extension 层的 runtime 委派 |
| 04 | [plan mode 不是内建 planner，而是宿主切出来的一种工作模式](docs/articles/04-pi-plan-mode-不是内建-planner-而是-runtime-mode-switch.md) | 为什么 plan mode 是主 runtime 的 mode switch |
| 05 | [当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么](docs/articles/05-pi-extension-surface-让-agent-产品变成宿主平台.md) | 为什么 extension surface 会把 agent 产品推向宿主平台 |
| 06 | [为什么 pi 代表另一种 agent 产品路线](docs/articles/06-pi-为什么代表另一种-agent-产品路线.md) | `pi` 适合谁、不适合谁，以及它和强成品 agent 的关系 |

---

## 配图索引

所有文章配图都放在：

- [`docs/assets/`](docs/assets/)
- [`docs/assets/INDEX.md`](docs/assets/INDEX.md)

目前每章至少有一张结构图：

- 00：阅读地图
- 01：Claude Code vs pi 路线对照图
- 02：session tree runtime 图
- 03：subagent runtime delegation 图
- 04：plan mode runtime switch 图
- 05：extension surface host platform 图
- 06：产品路线结语对照图

---

## 过程材料

### 规划与展开稿

- [`docs/guidebook/2026-04-23-pi-guidebook-scope-v1.md`](docs/guidebook/2026-04-23-pi-guidebook-scope-v1.md)
- [`docs/guidebook/2026-04-23-pi-book-outline-v1.md`](docs/guidebook/2026-04-23-pi-book-outline-v1.md)
- [`docs/guidebook/2026-04-23-pi-writing-cards-v1.md`](docs/guidebook/2026-04-23-pi-writing-cards-v1.md)

### 源码细读笔记

- [`docs/notes/2026-04-23-pi-vs-cc-差异点初判.md`](docs/notes/2026-04-23-pi-vs-cc-差异点初判.md)
- [`docs/notes/2026-04-23-pi-extension-runtime-初读.md`](docs/notes/2026-04-23-pi-extension-runtime-初读.md)
- [`docs/notes/2026-04-23-pi-session-runtime-初读.md`](docs/notes/2026-04-23-pi-session-runtime-初读.md)
- [`docs/notes/2026-04-23-pi-subagent-实现细读-v1.md`](docs/notes/2026-04-23-pi-subagent-实现细读-v1.md)
- [`docs/notes/2026-04-23-pi-plan-mode-实现细读-v1.md`](docs/notes/2026-04-23-pi-plan-mode-实现细读-v1.md)

---

## 当前状态

主体小书 `00-06` 已补齐。后续重点不再是继续加章节，而是：

1. 统一文件命名与章节顺序；
2. 清理过期规划文案；
3. 视需要补站点/导航/发布入口。
