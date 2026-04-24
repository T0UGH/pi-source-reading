# 2026-04-23 pi chapter 01 expansion v1

## 章节定位

这一章不是普通的“项目介绍”。

它其实承担整本书的总论点建立工作。

如果这一章写稳了，后面的 session tree、subagent、plan mode 都会变成自然展开；如果这一章写虚了，后面几章就容易被读成：
- pi 有哪些功能
- pi 也能做 subagent
- pi 也有 plan

那就会把整本书读歪。

所以 01 章真正要完成的，不是介绍 `pi`，而是先给读者一个稳定的判断框架：

> **`pi` 的亮点不是“功能又多了几项”，而是它把 coding agent 做成了一个可编程宿主。**

---

## 这一章要完成的 3 个任务

### 任务 1：把比较维度纠正过来
不要让读者继续用下面这些维度读：
- 命令多不多
- provider 多不多
- TUI 炫不炫
- 默认工作流全不全

而要改成：
- 高级工作流长在 core 里，还是长在 host surface 上
- 用户能不能不 fork 内核就扩 runtime
- session 能不能承载长期工作流
- extension 到底是插件点缀，还是正式 runtime surface

### 任务 2：先把 cc 和 pi 的路线差异钉死
这一章不追求“客观中立地列差异”，而要给出一个有判断力的 framing：

- **cc：强成品 / 内建工作流优先**
- **pi：宿主路线 / runtime surface 优先**

### 任务 3：给后面几章预埋入口
这一章结尾必须让读者自然接受：
- 既然 `pi` 是宿主
- 那就要继续看它的宿主底座是什么
- 它怎样长出委派型工作流
- 它怎样长出模态切换型工作流

也就是说，01 章必须是后面三章的发射台。

---

## 我建议的正文骨架

### 1. 开头先反常识：`pi` 最值得看的不是功能表
开头最好就直接破题：

> 如果把 `pi` 当作“又一个 terminal coding agent”来读，最后只能得到一份功能说明书；而它最值得研究的地方，恰好不是功能表，而是它重新定义了高级工作流该长在哪里。

这一段的作用：
- 把读者从功能视角拉走
- 建立“这本书不是大全”的阅读预期
- 给出整章的第一层 tension

### 2. 第一结论：cc 卖的是强成品，pi 卖的是宿主
这里要快速给结论，不要绕：

> Claude Code 更像一个已经做得很强的官方 agent 成品，而 `pi` 更像一个让你自己长出 agent 工作流的宿主。

然后马上解释“强成品”与“宿主”到底不是文学比喻，而是工程差异：

- 强成品：高级工作流主要由官方内建并维护
- 宿主：高级工作流主要通过 runtime surface 外化出来

### 3. 第一层证据：README 与 examples 已经把产品哲学写明了
这一节不要写太长，但必须稳。

重点抓两件事：
- `Pi is a minimal terminal coding harness`
- `Pi ships with powerful defaults but skips features like sub agents and plan mode`

这两句结合起来，能推出三个判断：

1. 它有意把自己定义成 harness，不是 all-in-one assistant
2. 它不是做不出 subagent / plan mode，而是故意不先做进 core
3. 它希望用户把高级工作流长在 extension/runtime 层

这一节的作用：
- 证明“宿主路线”不是我们主观发挥
- 而是项目自己公开表达的产品立场

### 4. 第二层证据：对照 cc 的 built-in workflow 路线
这一节是最重要的对照段之一。

可以直接对照：
- `builtInAgents.ts`
- `loadAgentsDir.ts`
- `forkSubagent.ts`

要讲清楚的不是“cc 也有这些能力”，而是：

- cc 的高级工作流更倾向成为 core product 的一部分
- worker、plan、verification、fork 都更像官方工作流骨架
- 它卖点的一部分就是“官方已经替你做强了”

建议这里写得稍微锋利一点，因为这是全书的主对照轴。

### 5. 第三层证据：`pi` 的 extension 不是插件点缀，而是正式 runtime surface
这一节应该是 01 章最硬的源码段。

需要把 extension surface 的层级列清楚：
- tools
- commands / shortcuts / flags
- UI components
- input / autocomplete / working indicator
- session lifecycle
- compaction
- provider registration

然后下判断：

> 当一个系统把这些层都开放给 extension 时，它已经不是“让你打补丁”，而是在让你把自己的 runtime 行为正式长进宿主。

这一节最好配一个强对比句：

- **插件感**：给产品补一点能力
- **平台感**：把自己的工作流长进宿主

### 6. 第四层证据：subagent 与 plan mode 都没有优先固化进 core
这一节不用把源码细节讲完，但要先把两条证据线打出去：

#### 6.1 subagent 线
先给一个短判断：
- 在 `pi` 里，subagent 不是官方内核里的固定角色系统，而是 extension 驱动的受控委派 runtime

点到这些关键动作就够：
- spawn 独立进程
- JSON mode 事件流
- agent-specific prompt/tool/model
- project-local agent trust boundary

#### 6.2 plan mode 线
再给一个短判断：
- 在 `pi` 里，plan 不是一个内建 worker，而是一种 extension 切出来的宿主运行模式

点到这些关键动作：
- 只读工具池
- allowlist
- before_agent_start 注入
- `[DONE:n]`
- custom entries / session resume

这一节的作用不是讲透实现，而是先告诉读者：

> `pi` 已经展示出至少两种不同的“高级工作流外化路线”。

### 7. 收成一句更硬的判断：`pi` 卖的是“工作流长在哪里”的重新定义
这一节要开始把整章收束。

可以明确写：

`pi` 最值得研究的，不是它“也能做哪些事”，而是它对这个问题给出了不同答案：

> 高级 agent 工作流，究竟应该长在官方核心里，还是长在宿主暴露出来的 runtime surface 上？

这句话最好成为整章的中轴句。

### 8. 结尾把后文发射出去
结尾别只做总结，要自然引到后三章：

- 如果这条判断成立
- 那后面最该看的就不是命令表
- 而是宿主底座（session tree）
- 委派型外化（subagent）
- 模态切换型外化（plan mode）

这样整本书的推进就顺了。

---

## 这一章可以进一步拆成的 4 个小问题

如果你想把 01 写得更亮，不一定只写成一篇大文；也可以把它内部拆成 4 个小问题来组织段落：

### 小问题 1
**为什么我说 `pi` 容易被“功能视角”读错？**

目的：先破错觉。

### 小问题 2
**为什么说 cc 和 pi 的真正差异，不在功能，而在工作流长在哪里？**

目的：建立总对照轴。

### 小问题 3
**为什么 extension runtime 才是 `pi` 的核心，而不是附属插件系统？**

目的：建立宿主论证的硬证据。

### 小问题 4
**为什么 subagent / plan mode 恰好证明了这条路线已经落地？**

目的：把抽象判断落回到具体实现。

---

## 我建议补强的一个结构：加一张“cc vs pi 路线对照表”

01 章很适合放一张高价值图或表。

比如：

| 维度 | Claude Code | pi |
|---|---|---|
| 产品定位 | 强成品 agent | 可编程宿主 |
| 高级工作流位置 | 优先内建进 core | 优先外化到 extension/runtime |
| plan | 内建 worker 倾向 | mode switch 倾向 |
| subagent | 官方 fork path | extension 驱动委派 runtime |
| 用户角色 | 使用强产品 | 在宿主上长工作流 |
| 主要卖点 | 官方已做强 | 不 fork 内核也能扩正式 runtime |

这张表的价值很高，因为它会让读者一下子明白：
- 不是“谁功能多一点”
- 而是“产品路线根本不同”

---

## 这一章最该避免的 5 个坑

### 坑 1：写成泛泛“产品定位分析”
一定要让每个判断都回到源码锚点，不然会飘。

### 坑 2：把 `pi` 吹成“比 cc 更先进”
这本书不该这么写。

更稳的写法是：
- 不是谁碾压谁
- 而是两条路线服务不同用户
- 但 `pi` 在“宿主化”这条路上更集中、更纯粹

### 坑 3：太早展开 session / subagent / plan mode 细节
01 章只要打出证据线，不要把后文内容吃掉。

### 坑 4：把 extension 写成普通插件机制
01 章必须把“插件感”和“平台感”分开。

### 坑 5：没有把“为什么值得研究”说透
不是只说它有意思，而是要说：
- 它回答了系统型用户真正关心的问题
- 也就是工作流边界、宿主边界、扩展边界长在哪里

---

## 这一章的推荐标题候选

如果想让 01 更亮一点，我建议备这几个标题版本：

### 标题候选 A
`pi` 不是强成品 agent，而是可编程宿主

这是当前标题，稳。

### 标题候选 B
Claude Code 在把 agent 做成产品，`pi` 在把 agent 做成宿主

这个更锋利，适合正文标题或导语大句。

### 标题候选 C
`pi` 最值得看的，不是功能表，而是它把工作流长成了宿主能力

这个更偏文章风。

---

## 我对 01 章的最终判断

如果 01 章只允许保留一个核心句，我建议就是这句：

> **`pi` 的差异点，不是它比 Claude Code 多做了什么功能，而是它把高级工作流从“官方内建能力”改成了“宿主可长出来的 runtime 能力”。**

这句比“可编程宿主”更进一步，因为它把“宿主”到底宿主了什么，说得更具体。

---

## 基于 01，后面最自然的展开顺序

如果按 01 来展开整本书，最自然的续写顺序就是：

1. **02：为什么 pi 要把 session 做成树，而不是聊天记录**
   - 证明宿主不是空话，而是有 runtime 底座
2. **03：pi 的 subagent 不是 prompt 分身，而是独立 runtime 的委派**
   - 证明委派型工作流如何外化
3. **04：plan mode 不是内建角色，而是宿主切出来的一种工作模式**
   - 证明模态切换型工作流如何外化
4. **05：当 tool、UI、provider、compaction 都能外化，agent 产品会变成什么**
   - 把整条路线收成完整产品判断

也就是说，01 章不是普通第一章，而是整本书的发动机。