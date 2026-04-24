# 2026-04-24 pi ch06 conclusion expansion v1

## 章节定位

第 06 章是整本小书的结语。

它不再继续展开新的源码细节，而是把 01-05 章的证据收成一个产品路线判断：

> `pi` 不是“另一个 Claude Code”，而是代表了另一种 agent 产品路线：少一点官方强成品，多一点宿主 surface；少一点固定流程，多一点可生长工作台。

这一章要短、稳、清楚。

---

## 正文应该回答的问题

### 1. 读完整本书后，应该怎样重新定位 `pi`？

不能把它定位成：

- 功能最全的 coding agent；
- 最省心的开箱产品；
- 官方替你定义完整工作流的 assistant；
- 直接替代 Claude Code 的强成品。

更合适的定位是：

> 一个 minimal 但 surface 足够强的 coding-agent host。

它的价值不在“当前功能最多”，而在“用户可以继续长出自己的工作流”。

---

### 2. 它不适合谁？

要讲清楚边界。

它不一定适合：

- 只想开箱即用的人；
- 不想理解 runtime / extension 的人；
- 不想承担权限、provider、workflow 治理的人；
- 希望官方把 planner / reviewer / subagent 都定义好的人。

这不是贬低 `pi`，而是路线选择。

---

### 3. 它特别适合谁？

它适合系统型用户：

- 想自己搭 agent 工作台；
- 想接内部 provider / proxy；
- 想把团队流程做进 extension；
- 想自定义 plan / review / subagent / compaction；
- 想把 session 当长期 runtime state 管理；
- 能接受更高工程复杂度，换取更高可塑性。

---

### 4. 它和 Claude Code / cc 的关系是什么？

不要写成谁替代谁。

更准确的关系是两条路线：

- Claude Code / cc：强成品 agent 路线，官方主流程更强，用户更省心；
- `pi`：可编程宿主路线，官方 core 更克制，extension surface 更重要。

结论：

> 它们不是简单 feature checklist 上的竞争，而是高级工作流应该长在哪里的路线分叉。

---

## 建议正文结构

### 标题

06｜为什么 pi 代表另一种 agent 产品路线

### 开头

回到全书最初的判断：如果按“又一个 coding agent”读，会读错 `pi`。

### 1. 五章证明了同一句话

用 5 个 bullet 收前文：

- 01：不是强成品，而是可编程宿主；
- 02：session tree 让工作过程成为 runtime state；
- 03：subagent 是 extension 层 runtime 委派；
- 04：plan mode 是主 runtime mode switch；
- 05：extension surface 让 agent 产品变成宿主平台。

### 2. pi 不是为了所有人

写边界。

### 3. pi 对系统型用户特别值钱

写它真正适合的人。

### 4. 它和强成品 agent 是两条路线

不要写谁赢谁输，而是路线差异。

### 5. 最后一锤

> `pi` 不是要赢在功能最多，而是把 agent 做成可继续生长的宿主。

---

## 建议配图

文件：`docs/assets/pi-ch06-product-route-conclusion.svg`

图要表达：

1. 左侧：强成品 agent 路线；
2. 中间：`pi` 的可编程宿主路线；
3. 右侧：适合的用户；
4. 底部一句话收束。

---

## 最该避免的写法

1. 不要继续引入太多新源码。
2. 不要写成广告稿。
3. 不要说 `pi` 一定更好；它只是更适合另一类用户。
4. 不要忘记代价：治理、权限、工程门槛。
5. 不要把 Claude Code 写成反面；它是另一条成熟路线。
