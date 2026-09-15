---
name: dispatching-parallel-agents
description: Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies
---

# Dispatching Parallel Agents

## Overview

你把任务委派给**上下文隔离**的专职 Agent。通过精确构造它们的指令与上下文，你保证它们聚焦、成功。它们**永远不应该继承你的会话上下文或历史**——你需要什么，就构造什么给它。这同时把你的上下文留给协调工作。

当你面对**多个互不相关的失败**（不同测试文件、不同子系统、不同 bug），顺序调查纯属浪费时间。每项调查彼此独立，可以同时发生。

**Core principle:** Dispatch one agent per independent problem domain. Let them work concurrently.

## When to Use

```
有多个失败/任务？
├─ 否 → 正常单线处理
└─ 是 → 它们互相独立吗？
        ├─ 否（相关：修一个可能顺带修掉别的）→ 单个 Agent 一起查
        └─ 是 → 能并行吗（无共享状态）？
                ├─ 是 → 并行派发（本 skill 的正题）
                └─ 否（共享状态/同改同一处）→ 串行派发
```

**Use when:**
- 3+ 测试文件失败，**根因各不相同**
- 多个子系统**各自独立**地坏掉
- 每个问题**不需要**别人的上下文就能理解
- 各项调查之间**无共享状态**

**Don't use when:**
- 失败之间相关（修一个可能修掉其他）——先合起来查
- 需要理解完整系统状态
- Agent 之间会互相干扰
- 你还在探索阶段，**根本不知道哪儿坏了**

## The Pattern

### 1. Identify Independent Domains

按"哪儿坏了"给失败分组：
- A 文件的测试：工具审批流程
- B 文件的测试：批量完成行为
- C 文件的测试：中止功能

每个域独立——修好工具审批不影响中止测试。

### 2. Create Focused Agent Tasks

每个 Agent 拿到：
- **Specific scope**：一个测试文件或一个子系统
- **Clear goal**：让这些测试通过
- **Constraints:** `Don't change other code`（**硬约束**——不写清，Agent 会顺手重构全世界）
- **Expected output**：你发现了什么、修了什么

### 3. Dispatch in Parallel

**在同一次响应里**发出全部派发调用——它们才会真正并行：

```text
Subagent: "Fix agent-tool-abort.test.ts failures"
Subagent: "Fix batch-completion-behavior.test.ts failures"
Subagent: "Fix tool-approval-race-conditions.test.ts failures"
# 三个同时跑。
```

**关键技巧：**
- **同一响应里发多个派发调用 = 并行执行**
- **分在不同响应里发 = 串行执行**（后面那批要等前一批，白白拉长总时长）

这是本 skill 最容易被搞错的一点：并行的开关不在"说一句请并行"，而在**它们是不是同一条消息里的多个调用**。

### 4. Review and Integrate

Agent 返回后：
- 读每份小结
- **检查修复之间有没有冲突**（两个 Agent 是不是改了同一段代码）
- 跑**全量**测试套件
- **抽查**（Agent 会犯系统性错误）
- 合并全部改动

## Agent Prompt Structure

好的 Agent prompt 有三个要素：
1. **Focused** - 一个清晰的问题域
2. **Self-contained** - 理解问题所需的**全部**上下文都在 prompt 里（不许让它来问你）
3. **Specific about output** - Agent 应该回传什么？

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" - fast tool aborted instead of completed
3. "should properly track pendingToolCount" - expects 3 results but gets 0

These are timing/race condition issues. Your task:

1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by:
   - Replacing arbitrary timeouts with event-based waiting
   - Fixing bugs in abort implementation if found
   - Adjusting test expectations if testing changed behavior

Do NOT just increase timeouts - find the real issue.
Do NOT change other code.

Return: Summary of what you found and what you fixed.
```

## Common Mistakes

**❌ Too broad:** "Fix all the tests" - agent gets lost
**✅ Specific:** "Fix agent-tool-abort.test.ts" - focused scope

**❌ No context:** "Fix the race condition" - agent doesn't know where
**✅ Context:** Paste the error messages and test names

**❌ No constraints:** Agent might refactor everything
**✅ Constraints:** "Do NOT change production code" or "Fix tests only"

**❌ Vague output:** "Fix it" - you don't know what changed
**✅ Specific:** "Return summary of root cause and changes"

## When NOT to Use

**Related failures:** 修一个可能顺带修掉其他——先一起查
**Need full context:** 必须看到整个系统才能理解
**Exploratory debugging:** 你还不清楚哪儿坏了，先自己定位问题域，再考虑并行
**Shared state:** Agent 会互相干扰（改同一批文件、抢同一个资源、占同一个端口）

## Real Example from Session

**Scenario:** 一次大重构后，3 个文件共 6 个测试失败

**Failures:**
- agent-tool-abort.test.ts: 3 failures（时序问题）
- batch-completion-behavior.test.ts: 2 failures（工具没被执行）
- tool-approval-race-conditions.test.ts: 1 failure（执行计数 = 0）

**Decision:** 三个独立域——中止逻辑、批量完成、竞态条件互不相干

**Dispatch:**
```
Agent 1 → Fix agent-tool-abort.test.ts
Agent 2 → Fix batch-completion-behavior.test.ts
Agent 3 → Fix tool-approval-race-conditions.test.ts
```

**Results:**
- Agent 1: 把超时换成基于事件的等待
- Agent 2: 修了事件结构 bug（threadId 放错位置）
- Agent 3: 补上等待异步工具执行完成的处理

**Integration:** 三处修复互不冲突，全量测试绿

## Verification

Agent 返回后：
1. **Review each summary** - 弄清每一处改了什么
2. **Check for conflicts** - Agent 之间有没有改到同一处代码？
3. **Run full suite** - 验证所有修复合在一起仍然成立
4. **Spot check** - Agent 会犯系统性错误，抽查几处

**第 2 步常被跳过，代价最高**：两个并行 Agent 各自"正确"地改了同一个函数时，单看每一份小结都合理，合起来才是坏的。

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 需要跑的是完整计划里的独立任务 | `subagent-driven-development`——比本 skill 更高一层的编排 |
| 还没定位问题域、不清楚哪儿坏了 | `systematic-debugging`——先查清根因，再决定能不能并行 |
| 并行 Agent 返回后**出现了冲突** | `systematic-debugging`——冲突是症状，先定根因再合 |
| 要派的是代码审查者 | `requesting-code-review`——审查**不要**并行派发多个（模板里明确禁止） |
| 需要隔离工作区来放并行产物 | `using-git-worktrees`——每个 Agent 一个 worktree，从物理上杜绝共享状态 |
| 各自改完、准备声称没问题 | `verification-before-completion`——合起来跑全量实证 |

**本 skill 管"独立问题域的并发派发"；它不管"任务循环怎么编排"（那是 `subagent-driven-development`），也不管"根因是什么"（那是 `systematic-debugging`）。**
