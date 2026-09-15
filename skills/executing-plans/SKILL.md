---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

载入计划，批判性评审，执行全部任务，完成后汇报。

**Announce at start:** "I'm using the executing-plans skill to implement this plan."
（开场自报家门：我正在使用 executing-plans skill 来实施这份计划。）

**Note:** 有子 Agent 能力时，用 `subagent-driven-development`，**不要用本 skill**。本 skill 是它的降级替代——子 Agent 委派机制不可用、或用户明确要求单会话执行时才用它。原因见下节。

## 与 subagent-driven-development 的分工差异

同一个计划，两种执行方式的分野只有一条：**实现和审查由谁来做。**

| | `subagent-driven-development` | `executing-plans`（本 skill） |
|---|---|---|
| 谁写代码 | 每个任务派一个全新子 Agent | 本会话自己写 |
| 上下文 | 实现者的上下文是**构造出来的**，不继承会话历史 | 实现和协调共用同一个上下文 |
| 任务间 review | 每任务两道裁决（spec 合规 + 代码质量），终局再做一次全分支 review | 无独立 review 环节，靠自己按 step 跑验证 |
| 出错的独立性 | 换一双新眼睛；4-5 轮还能换更强的模型 | 同一双眼睛看第二遍，盲区不会消失 |
| 迭代速度 | 任务之间不停下来问人 | 批量执行，遇阻塞才停 |
| 上下文占用 | 协调用的上下文保持干净 | 实现细节会持续占据会话上下文 |

**本 skill 的代价是明确的**：没有独立审查者，没有审查门。所以它只适合**计划本身足够细、验证足够硬**的场景——也就是计划里每个 step 都带了 `Run:` 和 `Expected:` 的情况。计划里全是"加上适当的错误处理"这种话，本 skill 会直接把它执行成不可验证的churn。

## The Process

### Step 1: Load and Review Plan

1. 确保有隔离工作区：用 `using-git-worktrees` 创建，或核实已有
2. 读计划文件
3. **批判性评审**——找出你对这份计划的所有疑问和顾虑
4. **有顾虑：先向用户提出，再动手**
5. 无顾虑：为计划中的各项建任务清单，然后往下走

第 3-4 步不是形式。计划可能写错了、可能过期了、可能漏了某个全局约束。你在这一步的怀疑是**最后一次廉价的机会**——一旦开始执行，同样的疑虑会以返工的形式变贵。

### Step 2: Execute Tasks

对每个任务：
1. 标记为进行中
2. **严格按 step 执行**（计划里已经是小到一口吃下的 step）
3. 按计划里的规定跑验证
4. 标记为已完成

### Step 3: Complete Development

所有任务完成并验证后：
- 宣告："I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** 使用 `finishing-a-development-branch`
- 按那个 skill 走：验证测试、列出选项、执行所选

## When to Stop and Ask for Help

**停下，立刻：**
- 撞上阻塞（缺依赖、测试失败、指令不清楚）
- 计划有关键缺口，根本起不了步
- 某条指令你看不懂
- 验证反复失败

**去问清楚，不要猜。**

注意本 skill **不是按固定间隔插检查点**——没有"每 3 个任务问一次"这种规则。任务的执行是连续的，**只有上面这些情况才打断它**。固定间隔的检查点会浪费用户的时间，而且通常在没有任何问题时打断；真正的信号是"卡住了"，那就以卡住为准。

## When to Revisit Earlier Steps

**回到 Step 1（评审）：**
- 用户根据你的反馈更新了计划
- 基本思路需要重新考虑

**不要硬冲阻塞** —— 停下，问。

## Remember

- 先批判性评审计划
- 严格按计划 step 走
- 不跳过验证
- 计划说要用哪个 skill 就用哪个
- 卡住就停，不要猜
- **没有用户明确同意，绝不在 main/master 分支上开始实现**

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| **有子 Agent 委派机制可用** | `subagent-driven-development`——本 skill 是它的降级替代，不要在有子 Agent 时用本 skill |
| 还没有计划 | 先转 `writing-plans`（没有 spec 则更早一步回创建期流程） |
| 执行中撞上阻塞、测试失败、根因不明 | `systematic-debugging`——不要在本 skill 里硬猜着改 |
| 写实现本身 | `test-driven-development`——计划里的 step 顺序就是它的循环 |
| 实现容易过度、冒出多余抽象 | `ponytail` |
| 准备宣告某项完成 | `verification-before-completion`——先验证再声称 |
| 全部任务完成 | `finishing-a-development-branch` |
| 需要隔离工作区 | `using-git-worktrees` |

**本 skill 管"单会话按计划执行"；有子 Agent 时它的正确去向是 `subagent-driven-development`。**
