# 2026-04-24 pi ch00 entry expansion v1

## 章节定位

00 是整本小书的总入口。

它不承担源码细读任务，而是负责把读者从“又一个 coding agent / 功能表对比”的阅读方式里拉出来。

它要先交代：

1. 为什么 `pi` 值得单独研究；
2. 为什么这本书不做大全；
3. 为什么“强成品 agent vs 可编程宿主”是正确入口；
4. 后面几章应该按什么顺序读。

核心判断：

> `pi` 值得单独研究，不是因为它比 Claude Code 多了什么功能，而是因为它把高级工作流应该长在哪里这件事重新打开了。

---

## 正文应该回答的问题

### 1. 为什么不能按“又一个 coding agent”读？

如果从 CLI、provider、命令、TUI、默认工具这些角度读，很容易把 `pi` 读成普通功能说明书。

但 README 第一段已经给出更关键的定位：

> Pi is a minimal terminal coding harness. Adapt pi to your workflows, not the other way around, without having to fork and modify pi internals.

这句话说明它不是强调“官方功能最多”，而是强调“适配你的 workflow”。

紧接着 README 又说：

> Pi ships with powerful defaults but skips features like sub agents and plan mode.

这说明它不是不知道 subagent / plan mode 重要，而是刻意不把它们做进 core。

---

### 2. 为什么这本书不做大全？

因为大全会稀释主线。

不写：

- 安装教程；
- provider 大全；
- TUI 教程；
- 命令大全；
- monorepo 全量漫游。

只回答三个问题：

1. `pi` 和 Claude Code 最大差异是什么？
2. `pi` 的真正卖点是什么？
3. 为什么它对会自己搭 agent 工作台的人值得研究？

---

### 3. 为什么“强成品 vs 可编程宿主”是入口？

因为差异不在单点功能，而在高级工作流的位置。

Claude Code / cc 更像强成品路线：

- 官方定义主流程；
- 高级工作流优先内建；
- 用户少操心；
- 产品路径更固定。

`pi` 更像可编程宿主路线：

- core 更克制；
- session 是 runtime state；
- subagent、plan mode 长在 extension 层；
- provider、compaction、UI 等基础层也可外化；
- 用户需要更多工程能力，但获得更高可塑性。

---

## 建议正文结构

### 标题

00｜这不是另一个 Claude Code：为什么 pi 值得单独研究

### 开头

直接排除误读：不要把它当作又一个 terminal coding agent 的功能表。

### 1. README 已经把路线说白了

引用 minimal terminal coding harness 与 skips sub agents / plan mode。

### 2. 这本书不做大全

说明范围边界。

### 3. 只回答三个问题

把 scope 文件中的 Q1/Q2/Q3 写进正文。

### 4. 阅读地图

按 01-06 说明每章作用。

### 5. 入口结论

> `pi` 的卖点不是功能更多，而是把 agent 做成可编程宿主。

---

## 建议配图

文件：`docs/assets/pi-ch00-reading-map.svg`

图表达：

1. 先排除误读；
2. 再提出三大核心问题；
3. 最后给出 01-06 的阅读顺序。

---

## 最该避免的写法

1. 不要重复 01 章的所有论证，00 只做入口。
2. 不要写太多源码细节。
3. 不要讲成 README 复述。
4. 不要把 Claude Code 写成反面；这里讲路线差异。
