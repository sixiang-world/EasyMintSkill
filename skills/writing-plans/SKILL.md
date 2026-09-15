---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

写一份完整的实施计划，假设读这份计划的工程师**对我们的代码库零上下文，且品味可疑**。把所有他们需要知道的东西写下来：每个任务要碰哪些文件、代码、测试、可能要查的文档、怎么验证。整份计划拆成一口能吃下的小任务。DRY。YAGNI。TDD。频繁提交。

假设读者是一名熟练的开发者，但**对我们的工具链和问题域几乎一无所知**。假设他们不太懂什么叫好的测试设计。

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."
（开场自报家门：我正在使用 writing-plans skill 来写实施计划。）

**Context:** 如果要在隔离工作区里干活，它应该在**执行时**由 `using-git-worktrees` skill 创建。

**Save plans to:** `.agentskill/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划存放位置有偏好时，以用户偏好为准）

## Scope Check

如果 spec 覆盖了多个互相独立的子系统，那它本应在创建期被拆成多个子项目 spec。如果没拆，建议把这份计划拆开——**一个子系统一份计划**。每份计划都应该能独立产出可用、可测的软件。

## File Structure

在定义任务之前，先把**哪些文件会被创建或修改、每个文件负责什么**摊开画一遍。分解决策就是在这里锁定的。

- 设计单元要有清晰边界和明确接口。每个文件应该只有一项明确职责。
- 你最能推理的是**能一次性装进上下文**的代码，文件聚焦时你的编辑也更可靠。宁要小而聚焦的文件，不要什么都干的大文件。
- 一起变化的文件应该住在一起。按**职责**切分，不要按技术分层切分。
- 在既有代码库里，遵循已确立的模式。如果既有代码库就是大文件，不要单方面重构——但如果你要改的文件已经臃肿得没法用了，在计划里安排一次拆分是合理的。

这个结构决定了任务怎么分解。每个任务都应该产出**独立成立**的自包含变更。

## Task Right-Sizing

一个任务，是**最小到能自带测试周期、值得让一名新审查者把关的单位**。

划任务边界时：把环境搭建、配置、脚手架、文档这几类步骤，**折叠进那个需要它们的交付物所属的任务**；只在"审查者可以合理地否掉一个任务、同时批准它旁边那个任务"的地方切一刀。每个任务都以**一个可独立验证的交付物**收尾。

## Bite-Sized Task Granularity

**每个 step 是一个动作（2-5 分钟）：**
- "写失败的测试" —— 一个 step
- "跑它，确认它失败" —— 一个 step
- "写让测试通过的最少代码" —— 一个 step
- "跑测试，确认通过" —— 一个 step
- "提交" —— 一个 step

## Plan Document Header

**每份计划都必须以这个 header 开头：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use subagent-driven-development (recommended) or executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

> 中文对照：
> `For agentic workers` 那一行是给执行者的**路由提示**——计划一旦落盘，读者就是 agent，这行告诉他们用哪个 skill 执行、step 用 `- [ ]` 复选框跟踪进度。
>
> `Goal` 一句话说清这份计划建的是什么。`Architecture` 两三句说清思路。`Tech Stack` 列关键技术/库。
>
> `Spec` 是这份计划所实现的 spec/设计文档路径——**计划是从 spec 论证出来的，所以 spec 要跟着计划走**，执行者两份都读。
>
> `Global Constraints` 是 spec 的**项目级**要求（版本下限、依赖限制、命名与文案规则、平台要求），**一行一条，精确值从 spec 原样抄过来**。每个任务的要求都**隐含包含这一节**。

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

> 中文对照——每个任务的必备字段：
>
> **`Files`**：`Create` / `Modify` / `Test` 三类，路径写全，**Modify 要带精确行号范围**（`existing.py:123-145`）。"改一下那个文件"不算信息。
>
> **`Interfaces`**：`Consumes` 写明本任务从**前面任务**那里用到什么（精确签名）；`Produces` 写明**后面任务**会依赖什么（精确函数名、参数与返回类型）。理由：一个任务的实现者**只看得到自己那一个任务**，这个 block 就是他们得知邻居任务在用什么名字、什么类型的唯一渠道。
>
> **每个 Step**：要带 `Run:`（跑哪条命令）和 `Expected:`（期望什么输出）。只说"跑测试"而不给命令与期望结果，等于没写。
>
> **代码 step 必须带代码块**——描述"该做什么"但不给代码，是计划失败，见下节。

## No Placeholders

每个 step 都必须包含工程师真正需要的**实际内容**。以下这些是**计划失败**，永远不要写：

- "TBD"、"TODO"、"later 再实现"、"细节待补"
- "加上适当的错误处理" / "加上校验" / "处理边界情况"
- "为上面的内容写测试"（却不给实际测试代码）
- **"与 Task N 类似"**（把代码重复一遍——**工程师可能会乱序阅读任务**）
- 只描述该做什么、不展示怎么做的 step（代码 step 必须给代码块）
- 引用了任何任务里都没定义的 type、function 或 method

## Self-Review

计划写完后，用一双新眼睛看 spec，拿计划对着 spec 核一遍。这是**你自己跑的清单**，不是派子 Agent 的活。

**1. Spec coverage:** 逐节逐条扫一遍 spec 的每个 section/requirement。你能指出哪个任务实现了它吗？把缺口列出来。

**2. Placeholder scan:** 在计划里搜上节"执行失败"的红旗模式——上面 "No Placeholders" 那一节里的每一种。发现就修。

**3. Type consistency:** 你在后面任务里用的类型、方法签名、属性名，和你在前面任务里定义的一致吗？Task 3 里叫 `clearLayers()`、Task 7 里叫 `clearFullLayers()` —— 这是个 bug。

发现问题就**就地修掉**。不需要再审查一轮——修完继续。如果发现某条 spec 要求没有对应任务，把任务补上。

## Execution Handoff

计划保存后，把执行方式的选择权交给用户：

**"计划已完成，保存到 `.agentskill/plans/<filename>.md`。两种执行方式：**

**1. Subagent-Driven（推荐）** - 每个任务派一个全新子 Agent，任务之间做 review，迭代快

**2. Inline Execution** - 在本会话内用 executing-plans 执行任务，批量执行

**选哪种？"**

**如果选了 Subagent-Driven：**
- **REQUIRED SUB-SKILL:** 使用 `subagent-driven-development`

**如果选了 Inline Execution：**
- **REQUIRED SUB-SKILL:** 使用 `executing-plans`

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 还没有 spec / 需求还很模糊 | 先回到创建期流程（`creation-guide` 及 `creation-flow-*`）——计划从 spec 论证出来，没有 spec 就没有可论证的前提 |
| spec 覆盖多个独立子系统 | 先拆分 spec，再一个子系统一份计划 |
| 计划写完，选择 subagent 方式执行 | `subagent-driven-development`——计划文档本身是它的输入 |
| 计划写完，选择单会话方式执行 | `executing-plans` |
| 计划里某个任务是"必须先搞清根因"的调试 | 那个任务注明走 `systematic-debugging`，不要把猜的修法写进 step |
| 用例怎么切、测试怎么写 | `test-driven-development`——本 skill 的 step 结构就是按它的循环铺的 |
| 任务拆得太细、到处都是用不上的抽象 | `ponytail`——计划阶段就该拦掉过度工程 |
| 需要隔离工作区 | `using-git-worktrees`（执行时创建，不是写计划时） |

**本 skill 管"计划写什么"；计划写完不要自己开始实现——把上面那个二选一交给用户。**
