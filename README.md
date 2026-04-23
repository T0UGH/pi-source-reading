# pi-source-reading

> 一套围绕 **pi 为什么不是另一个 Claude Code，而是一种可编程 coding-agent 宿主** 展开的源码导读项目。

这个仓库不做 `pi` 的大而全说明书。

当前主线只回答三个问题：

1. `pi` 和 Claude Code 最大的差异点是什么
2. `pi` 真正的卖点是什么
3. 为什么它对会自己搭 agent 工作台的人特别值得研究

## 当前入口

第一轮不按“CLI / tools / TUI / settings”这种通用目录来读。

而是先抓四条最有价值的主线：

1. **Extension Runtime**
2. **Tree Session Runtime**
3. **Subagent as isolated runtime delegation**
4. **UI / provider / compaction hostification**

## 当前文档

### 入口与结构
- `docs/guidebook/2026-04-23-pi-guidebook-scope-v1.md`
- `docs/guidebook/2026-04-23-pi-writing-cards-v1.md`

### 第一轮源码阅读笔记
- `docs/notes/2026-04-23-pi-vs-cc-差异点初判.md`
- `docs/notes/2026-04-23-pi-extension-runtime-初读.md`
- `docs/notes/2026-04-23-pi-session-runtime-初读.md`

## 当前判断

一句话说：

> `pi` 的卖点，不是它比 cc 多了什么功能，而是它把 agent 做成了可编程宿主。

这意味着它最值得读的不是“会不会调工具”，而是：

- 工作流为什么不内建进 core
- extension runtime 为什么被做成第一公民
- session 为什么被做成树
- subagent 为什么通过独立 `pi` 进程完成隔离委派

## 下一步

当前最优先推进顺序：

1. 写第一篇正文：`pi 不是强成品 agent，而是可编程宿主`
2. 继续补 `extension runtime` 与 `cc built-in workflow` 的对照证据
3. 再进入 tree session runtime
4. 最后写 subagent 路线
