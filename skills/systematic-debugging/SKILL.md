---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.
（核心原则：永远先找到根因再动手修。修症状就是失败。）

**Violating the letter of this process is violating the spirit of debugging.**
（违反这套流程的字面要求，就是在违反调试的精神。）

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

没走完 Phase 1，你就没有资格提出修复方案。

## When to Use

任何技术问题都用：

- 测试失败
- 生产环境 bug
- 行为不符合预期
- 性能问题
- 构建失败
- 集成问题

**Use this ESPECIALLY when:**
- 时间压力大（紧急情况最容易诱人猜）
- "就一个快速修复"看起来明摆着
- 你已经试过好几次修复了
- 上一次修复没起作用
- 你并没有完全理解这个问题

**Don't skip when:**
- 问题看起来简单（简单 bug 也有根因）
- 你很赶（赶工保证返工）
- 老板/用户现在就要（系统化比乱撞快）

## The Four Phases

每一阶段都必须完成后才能进入下一阶段。

### Phase 1: Root Cause Investigation

**在尝试任何修复之前：**

1. **Read Error Messages Carefully**
   - 不要跳过错误和警告
   - 它们经常直接包含答案
   - 完整读完堆栈
   - 记下行号、文件路径、错误码

2. **Reproduce Consistently**
   - 能稳定触发吗？
   - 确切的复现步骤是什么？
   - 每次都发生吗？
   - 如果不能复现 → 继续收集数据，不要猜

3. **Check Recent Changes**
   - 什么改动可能导致了这个？
   - `git diff`、最近的提交
   - 新依赖、配置变更
   - 环境差异

4. **Gather Evidence in Multi-Component Systems**

   **当系统有多个组件时（构建链、API → 服务 → 数据库、客户端 → 网关 → 后端）：**

   **在提出修复方案之前，先加诊断埋点：**
   ```
   对每一个组件边界：
     - 记录进入该组件的数据
     - 记录离开该组件的数据
     - 验证环境/配置是否传递到位
     - 检查每一层的状态

   先跑一次，拿到"哪里断"的证据
   再分析证据，定位出问题的组件
   然后只查那一个组件
   ```

   **示例（多层系统）：**
   ```bash
   # 第 1 层：工作流
   echo "=== 工作流里可见的凭据变量: ==="
   echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

   # 第 2 层：构建脚本
   echo "=== 构建脚本里的环境变量: ==="
   env | grep IDENTITY || echo "IDENTITY 不在环境里"

   # 第 3 层：签名脚本
   echo "=== 密钥链状态: ==="
   security list-keychains
   security find-identity -v

   # 第 4 层：实际签名
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **这能揭示：** 是哪一层坏了（凭据 → 工作流 ✓，工作流 → 构建 ✗）

5. **Trace Data Flow**

   **当错误出现在调用栈深处时：**

   完整的上溯追源技法见 [references/root-cause-tracing.md](references/root-cause-tracing.md)。

   **简版：**
   - 坏值是从哪里产生的？
   - 谁带着坏值调用了这里？
   - 一直往上追到源头
   - 在源头修，不在症状处修

### Phase 2: Pattern Analysis

**先找到模式再修：**

1. **Find Working Examples**
   - 在同一个代码库里找类似的、能正常工作的代码
   - 有什么正常工作的东西和坏掉的很像？

2. **Compare Against References**
   - 如果在实现某个模式，把参考实现**完整**读完
   - 不要略读——每一行都读
   - 完全理解这个模式之后才应用

3. **Identify Differences**
   - 正常的和坏掉的区别在哪？
   - 把每一处差异都列出来，无论多小
   - 不要假定"那个不可能有影响"

4. **Understand Dependencies**
   - 它还依赖哪些其他组件？
   - 需要什么设置、配置、环境？
   - 它假设了什么？

### Phase 3: Hypothesis and Testing

**科学方法：**

1. **Form Single Hypothesis**
   - 明确写下来："我认为 X 是根因，因为 Y"
   - 写下来，别只在脑子里想
   - 要具体，不要含糊

2. **Test Minimally**
   - 做**最小**的改动来验证假设
   - 一次只动一个变量
   - 不要同时修多件事

3. **Verify Before Continuing**
   - 起作用了？→ 进入 Phase 4
   - 没起作用？→ 提出**新的**假设
   - 不要在失败之上再叠修复

4. **When You Don't Know**
   - 直接说"我不理解 X"
   - 不要装作知道
   - 求助于用户或外部资料
   - 去查更多资料

### Phase 4: Implementation

**修根因，不修症状：**

1. **Create Failing Test Case**
   - 尽可能最简单的复现
   - 能自动化测试就自动化
   - 没有测试框架就写一次性脚本
   - 修之前**必须**有
   - 用 `test-driven-development` skill 来写规范的失败测试

2. **Implement Single Fix**
   - 只针对已定位的根因
   - 一次只改**一处**
   - 不做"顺手也改一下"的改进
   - 不夹带重构

3. **Verify Fix**
   - 测试现在过了吗？
   - 有没有别的测试被搞挂？
   - 问题真的解决了吗？
   - 声称成功之前用 `verification-before-completion` skill

4. **If Fix Doesn't Work**
   - STOP
   - 数一数：你已经试了几次修复？
   - 少于 3 次：回到 Phase 1，带着新信息重新分析
   - **大于等于 3 次：STOP，质疑架构（见下面第 5 条）**
   - **不许在没有架构层讨论的情况下试第 4 次修复**

5. **If 3+ Fixes Failed: Question Architecture**

   **说明是架构问题的模式：**
   - 每修一次，就在另一个地方暴露出新的共享状态/耦合/问题
   - 修复需要"大规模重构"才能落地
   - 每修一次都在别处制造新症状

   **STOP，质疑根本假设：**
   - 这个模式本身站得住脚吗？
   - 我们是不是"靠惯性在硬撑"？
   - 应该重构架构，还是继续修症状？

   **在尝试更多修复之前，先和用户讨论**

   这不是一次失败的假设——这是**架构错了**。

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Pattern says X but I'll adapt it differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when already tried 2+)**
- **Each fix reveals new problem in different place**

**ALL of these mean: STOP. Return to Phase 1.**

**If 3+ fixes failed:** Question the architecture (see Phase 4.5)

> 以上任何一条出现，含义都是：停下，回到 Phase 1。已经失败 3 次以上，去质疑架构，不要再试。

## 用户发出的"你做错了"信号

**留意这些纠偏：**
- "Is that not happening?" —— 你没验证就假设了
- "Will it show us...?" —— 你本该先加证据采集
- "Stop guessing" —— 你在没理解的情况下提修复方案
- "Ultra-think this" —— 质疑根本假设，不是只质疑症状
- "We're stuck?"（带着情绪）—— 你的做法不管用

**看到这些：** STOP。回到 Phase 1。

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |

> 中文要点：问题简单也得走流程，简单 bug 走流程很快；紧急时系统化比乱撞快；第一个修复会定下模式，一开始就做对；没测试的修复站不住；一次改多处无法归因、还会引入新 bug；参考实现读一半等于保证出 bug；看到症状不等于理解根因；失败 2 次以上还要"再试一次"，其实是架构问题。

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

> **1. 根因调查**：读错误、稳定复现、检查最近改动、采集证据 → 成功标准：搞清楚"是什么"和"为什么"
> **2. 模式分析**：找能正常工作的例子并对比 → 成功标准：找出差异清单
> **3. 假设验证**：提出假设、最小验证 → 成功标准：假设被确认，或提出新假设
> **4. 实施修复**：写测试、修、验证 → 成功标准：bug 解决，测试全过

## When Process Reveals "No Root Cause"

如果系统化调查之后发现，问题确实是环境性、时序相关或外部导致的：

1. 你已经完成了流程
2. 把你查过什么记录下来
3. 实施恰当的应对（重试、超时、错误提示）
4. 加上监控/日志，方便以后再查

**But:** 95% of "no root cause" cases are incomplete investigation.
（但："没有根因"的情况里，95% 是调查没做完。）

## 归档技巧（本 skill 目录下）

- **[references/root-cause-tracing.md](references/root-cause-tracing.md)** —— 顺着调用栈上溯，找到最初的触发点
- **[references/defense-in-depth.md](references/defense-in-depth.md)** —— 找到根因之后，在多层加校验
- **[references/condition-based-waiting.md](references/condition-based-waiting.md)** —— 用条件轮询替换随意的时间等待

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 定位到根因，要修根因 | `test-driven-development`——先写复现测试，再写最小修复 |
| 修复完成，要声称"修好了" | `verification-before-completion`——**强制**，本 skill 的 Phase 4.3 已经要求它 |
| 3 次以上修复都失败，怀疑架构 | 停下，先和用户讨论架构；不要在本 skill 里继续试第 4 次 |
| 修复本身需要动较大的设计 | `ponytail`——判断是不是过度设计；必要时回到设计阶段对应 skill |
| 修完想审一遍代码质量 | `ponytail-review` |

**本 skill 管"怎么找到根因"；根因找到后修的部分回到 `test-driven-development`，声称完成前必须过 `verification-before-completion`。**
