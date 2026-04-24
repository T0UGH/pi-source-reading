# 源码细读笔记索引

这里放的是写作前后的源码细读笔记。它们保留阅读痕迹和源码锚点，不是最终正文。

最终正文入口见：[`../articles/INDEX.md`](../articles/INDEX.md)。

## 差异点与总判断

- [`2026-04-23-pi-vs-cc-差异点初判.md`](2026-04-23-pi-vs-cc-差异点初判.md)  
  初步判断 `pi` 与 Claude Code 的核心差异：不是功能多少，而是产品路线不同。

## runtime / session / extension

- [`2026-04-23-pi-extension-runtime-初读.md`](2026-04-23-pi-extension-runtime-初读.md)  
  初读 extension runtime，确认 extension 已经介入 tool、UI、session lifecycle 等宿主能力。
- [`2026-04-23-pi-session-runtime-初读.md`](2026-04-23-pi-session-runtime-初读.md)  
  初读 session runtime，确认 session tree 是长期 runtime state 的底座。

## 具体能力细读

- [`2026-04-23-pi-subagent-实现细读-v1.md`](2026-04-23-pi-subagent-实现细读-v1.md)  
  细读 subagent extension，确认它不是 prompt 分身，而是独立 `pi` 进程的 runtime 委派。
- [`2026-04-23-pi-plan-mode-实现细读-v1.md`](2026-04-23-pi-plan-mode-实现细读-v1.md)  
  细读 plan mode extension，确认它不是内建 planner，而是通过 active tools 与 session state 切换出来的工作模式。

## 当前状态

这些笔记中的早期“继续读什么”已经改写为完成状态说明；主体小书已按 `00-06` 收束完成。
