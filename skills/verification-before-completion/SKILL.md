---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs, or when about to report that a delegated sub-agent succeeded - requires running verification commands and confirming their output before making any success claim
---

# Verification Before Completion

## Overview

**Core principle:** Evidence before claims, always.
（核心原则：先有证据，再下结论，永远如此。）

**Violating the letter of this rule is violating the spirit of this rule.**
（违反这条规则的字面要求，就是在违反它的精神。）

这条规则针对的失败模式非常具体：Agent 在成功叙事上比在验证上更"顺滑"。改写一个文件、跑了一半检查、或者子 Agent 回报一句 success，语言系统就已经准备好说"已完成"了。本 skill 的作用就是在"想说话"和"说出口"之间插一道闸门。

**Announce at start:** "I'm using the verification-before-completion skill to verify this work before claiming it is done."
（开场自报家门：我正在使用 verification-before-completion skill，在声称工作完成之前先做验证。）

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

> 没有新鲜的验证证据，就不许做任何完成性声明。

**Fresh** 是关键限定词：证据必须来自**本次消息**、对着**当前这棵树**跑出来的。上次会话跑过的绿灯、改动前跑过的绿灯、子 Agent 口头回报的绿灯，统统不算。

如果你没有在这一轮里跑过验证命令，你就**没有资格**说它通过。

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

> 中文对照：
> 1. **IDENTIFY** 识别——哪条命令能证明这个说法？说不出来，就说明这个说法不该出口。
> 2. **RUN** 执行——跑**完整**命令，新鲜的、不裁剪的。
> 3. **READ** 读取——读完整个输出，看退出码，数失败条数。
> 4. **VERIFY** 核对——输出支持这个说法吗？不支持就如实说当前状态并附证据；支持才带着证据去说。
> 5. **ONLY THEN** 此时才——到这一步才允许开口。
>
> 跳过任何一步，你做的就不是验证，而是撒谎。

**"完整命令"的含义**：跑全量测试，不跑单个文件；跑整个构建，不跑 lint 替代；跑完整套要求清单，不是抽查两项。裁剪过的验证只能支撑裁剪过的结论——而你说出口的是完整结论。

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

> 中文对照（左列是"想说的话"，中列是"必须拿出的证据"，右列是"看起来像证据但其实不是的东西"）：
> - "测试全过"需要测试命令输出 0 失败；上一次的运行结果、"应该能过"不算。
> - "lint 干净"需要 lint 输出 0 错误；局部检查、外推不算。
> - "构建通过"需要构建命令退出码 0；lint 过了、日志看着对不算。
> - "bug 修好了"需要复现原始症状的测试通过；改了代码、假定修好不算。
> - "回归测试有效"需要红-绿循环被验证过；测试恰好过一次不算。
> - "子 Agent 完成了"需要版本控制 diff 上真的有改动；子 Agent 报告"success"不算。
> - "需求都满足了"需要逐条对照检查清单；测试通过不算。

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

> 中文对照（出现任意一条，STOP）：
> - 用"应该""大概""似乎"。
> - 验证之前就先表达满意（"太好了""完美""搞定"等）。
> - 还没验证就准备提交 / push / 开 PR。
> - 相信子 Agent 的成功报告。
> - 依赖局部验证。
> - 冒出"就这一次"的念头。
> - 累了，想让活赶紧结束。
> - **任何暗示成功的措辞，而验证命令还没跑过。**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

> 中文对照：
> - "现在应该能跑了" → 去跑验证。
> - "我有把握" → 把握不等于证据。
> - "就这一次" → 没有例外。
> - "lint 过了" → lint 不等于编译。
> - "Agent 说成功了" → 独立验证。
> - "我累了" → 疲惫不是理由。
> - "查了一部分够了" → 局部证明不了任何事。
> - "我只是换了个说法，规则管不到" → 精神高于字面。

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

> 中文对照：
> - **测试类**：跑测试命令、看到 34/34 通过，才能说"测试全过"；"应该能过""看着是对的"不行。
> - **回归测试（TDD 红-绿循环）**：写测试 → 跑（通过）→ 把修复撤掉 → 跑（**必须失败**）→ 恢复 → 跑（通过）。只说"我写了回归测试"、没做过红-绿验证，不行。
> - **构建类**：跑构建、看到退出码 0，才能说"构建通过"。"lint 过了"不行——lint 不检查编译。
> - **需求类**：重读计划 → 建检查清单 → 逐条验证 → 报告缺口或完成。只说"测试过了，这个阶段完了"不行。
> - **委派类**：子 Agent 报成功 → 看版本控制 diff → 核实改动 → 报告真实状态。直接采信 Agent 的报告，不行。

**About "Agent delegation":** 子 Agent 的成功报告是**线索**，不是**证据**。子 Agent 和你一样会陷入"顺滑叙事"，而且它更缺少全局视角。验证方式是独立看 diff：文件真的变了吗？变的是不是该变的？声称跑过的测试，输出在哪里？如果拿不到输出，就自己重跑一遍。

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

> 中文对照：
> **以下情况永远适用**：任何形式的成功 / 完成声明；任何满意的表达；任何关于工作状态的正面陈述；提交、开 PR、任务收尾；进入下一个任务；委派子 Agent 之前。
>
> **规则覆盖范围**：原话本身；换一种说法和同义词；暗示成功；任何让人以为"完成了 / 正确了"的表达。

**为什么连"暗示"也管**：这条规则要拦的不是某个词，而是那种"事情已经好了"的语气。只换个说法，语气照旧，绕过的是字面，违反的是精神。

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 准备声称"修好了""测试过了""可以提交了" | **本 skill（强制）**——先跑验证命令，再开口 |
| 定位到根因、开始动手修 | `systematic-debugging`——本 skill 是它的 Phase 4.3 收口 |
| 正在写实现、还没写完 | `test-driven-development`——本 skill 不是它的替代品，是它的出口检查 |
| 验证通过，要决定怎么合入 | `finishing-a-development-branch`——其 Step 1 的绿测基线就是本 skill 的产出 |
| 需要独立判断改动是否真发生了 | 看版本控制 diff（`git diff` / `git status` / `git log`） |

**本 skill 是所有 skill 的收口**——不论前面走的是调试、TDD、原型还是重构，只要准备下"完成了"这个结论，就必须过本 skill 的闸门。**没有经过本 skill 的"完成"，只是"我以为完成"。**

**下游**：本 skill 给出绿测证据后，`finishing-a-development-branch` 才能开始走合入菜单。
