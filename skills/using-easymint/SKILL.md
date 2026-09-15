---
name: using-easymint
description: Use when starting any conversation, before responding to any request, or whenever about to write code, plan work, or fix a problem - establishes the mandatory rule that relevant skills must be invoked before any response or action, including clarifying questions.
---

<SUBAGENT-STOP>
若你是被委派来执行某个具体任务的子 Agent，忽略本 skill。你只需要读你的任务简报并执行。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
只要你认为某个 skill 有 **1%** 的可能适用于你正在做的事，你就 **必须** 调用它。

**如果一个 skill 适用于你的任务，你没有选择权。你必须使用它。**

这一点不可协商。你不能靠"想出一套说辞"绕过它。
</EXTREMELY-IMPORTANT>

## Overview

**核心原则**：任何回应或动作之前，先调用相关 skill。

这条规则的目的是解决一个真实且反复出现的失败模式——Agent 在压力下（时间紧、"任务很简单"、用户催）会绕过既定流程，直接凭直觉行动，最终产生返工。

## The Rule

**在做出任何回应或动作之前，先调用相关或用户指定的 skill**——包括澄清性提问、翻代码、看文件。

流程是：

1. 判断当前请求是否命中任何 skill 的触发条件（含 1% 可能）。
2. 命中则调用，并按该 skill 执行。
3. 在回应中声明：`Using [skill 名] to [目的]`。
4. 如果该 skill 带检查清单，**为每个条目建一个 todo**。
5. 若调用后发现不适用，可以不用——但调用动作必须先发生。

**进入计划模式之前**：若尚未做需求澄清，先调用 `creation-flow-intent` 或 `creation-guide`。

**委派子 Agent 之前**：先读 `easymint-core` 的 `references/03-orchestration.md`。

## 两层入口

本 skill 是**调度层**，`kickoff` 是**创作起手层**。两者都要用：

| 层 | skill | 触发时机 | 职责 |
|---|---|---|---|
| 调度层 | `using-easymint`（本 skill） | 任何对话开始、任何请求之前 | 判断该加载哪个 skill |
| 创作起手层 | `kickoff` | **任何创作性工作之前** | 分类规模 → 澄清需求 → 呈现设计 → 拿批准 → 派发 |

**任何"要做出新东西"的请求**（新功能、新项目、新组件、改行为）—— 先经 `kickoff` 分类和澄清，**拿到用户批准后**，再进入后续流程。

## Skill Priority（多个 skill 同时命中时）

**流程类 skill 优先于实现类 skill**——流程类定方法，实现类负责执行。

| 请求 | 先调用 | 再调用 |
|---|---|---|
| "帮我做个 X" | `creation-guide`（判复杂度、路由） | 对应 `creation-flow-*` |
| **任何创作性请求（新功能/新项目/改行为）** | **`kickoff`（分类规模 + 澄清 + 拿批准）** | **`creation-guide` 或 `writing-plans`** |
| "修一下这个 bug" | `systematic-debugging` | 领域 skill |
| "给这个功能写实现" | `test-driven-development` | 语言/框架 skill |
| "这代码写得好吗" | `ponytail-review` | — |

## Red Flags — 出现这些念头就停下

这些念头说明你正在给自己找借口。**它们的出现本身就是信号：先查 skill。**

| Thought | Reality |
|---|---|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "I can check git/files quickly" | Files lack conversation context. Check for skills. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Skills prevent this. |
| "I know what that means" | Knowing the concept ≠ using the skill. Invoke it. |
| "用户很急，先动手" | 急着动手正是 skill 存在的理由；返工比重来更慢。 |
| "我已经做过很多次这种任务了" | 熟练带来的是自信，不是免检。流程照走。 |

**All of these mean: Stop. Check for skills first.**

### Common Rationalizations

| Excuse | Reality |
|---|---|
| "任务太简单了，不用走流程" | 简单任务的假设错误代价最大。随复杂度缩减的是**产出物**，不是**校验**。 |
| "先做一个小的，再补流程" | 第一个动作会定下模式。要规范就从头规范。 |
| "用户没要求走流程" | 用户不要求流程，是因为他们不知道流程能省他们多少时间。 |
| "先探索一下再决定用什么 skill" | skill 就是告诉你该怎么探索的。先查。 |
| "我记得这个 skill 的内容" | skill 会更新。读当前版本。 |
| "这个 skill 和我的场景不完全一样" | 部分匹配也要读，读了你才能判断哪里不一样。 |

## 用户指令的优先级

优先级：**用户的明确指令** > **skill** > **默认行为**。

`AGENTS.md`、`CLAUDE.md`、项目说明文件、用户的直接要求，都优先于 skill。只有用户**明确说**跳过时，才可以跳过某个 skill 流程。

注意：用户说"快点"、"别搞太复杂"**不等于**"跳过流程"——那是在要求你更高效地执行流程。

## 平台适配（可选）

本包所有 skill 都是平台中立的，不绑定任何特定运行时的工具名。落地时按下表映射到你的环境：

| 方法论概念 | 通用实现 |
|---|---|
| 调用 skill | 读取该 skill 的 `SKILL.md` 全文（多数运行时把 skill 作为工具或在列表中暴露） |
| 委派子 Agent | 运行时的子 agent / 子会话机制；无此机制则串行执行并显式说明降级 |
| 任务清单 / todo | 运行时的任务工具；无则用文件（`.agentskill/tasks.md`） |
| 结构化选择题 | 文本列出 2-4 个选项请用户选择 |
| 打开原型预览 | 用默认浏览器打开本地 HTML 文件 |
| 图片理解 / 网页抓取 | 运行时的对应工具；无则请用户提供文字描述 |

**若你的环境在工具能力上有缺口，先如实说明降级方式，再继续——不要假装工具存在。**

## 与其他 skill 的关系

本 skill 是**入口层**，不承载具体方法论。执行任何任务时的完整链路：

```
using-easymint（本 skill，调度层：判断用哪个）
  └→ kickoff（创作起手层：分类 Spike/Bounded/Architectural → 澄清 → 设计 → 拿批准）
       │
       ├─ Spike ──────────→ 探针调查 → 汇报建议（终点）
       │
       ├─ Bounded ────────→ 用户批准简短设计 → 常规开发流程（TDD 照常，无计划文档）
       │
       └─ Architectural ──→ 写规格 → 规格自审 → 用户评审 → writing-plans
            └→ writing-plans（计划期：2-5 分钟粒度任务拆解）
                 └→ subagent-driven-development（执行期：任务循环 + 两阶段 review）
                      ├→ test-driven-development（实现纪律）
                      ├→ systematic-debugging（出了 bug）
                      ├→ requesting-code-review / receiving-code-review（审查循环）
                      └→ verification-before-completion（声称完成前）
                           └→ finishing-a-development-branch（收尾）
```

创建期（`creation-guide` / `creation-flow-*` 的 7 Gate 引导）可以**替代或补充** Architectural 路径的前半段 —— 当创建期流程已经产出了规格与任务清单时，从 `writing-plans` 或直接进入执行期即可，不必重复走 `kickoff` 的澄清阶段。**但批准闸门不可省。**
