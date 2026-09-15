---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review

派发一个代码审查子 Agent，在问题级联放大之前抓住它。审查者拿到的是**精确构造过的上下文**——永远不是你的会话历史。

**Core principle:** Review early, review often.

**Announce at start:** "I'm using the requesting-code-review skill to review this work."
（开场自报家门：我正在使用 requesting-code-review skill 来审查这项工作。）

## When to Request Review

**Mandatory（必做，不是可选项）：**
- `subagent-driven-development` 里**每个任务**完成之后
- 大功能完成之后
- 合并进 main 之前

**Optional but valuable（可选但有价值）：**
- 卡住的时候——借一双新眼睛
- 重构之前——先取基线
- 修完一个复杂 bug 之后

## How to Request

**1. 取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码审查子 Agent：**

按 [references/code-reviewer.md](references/code-reviewer.md) 的模板填充后派发。

**占位符：**
- `[DESCRIPTION]`——你构建了什么，一两句
- `[PLAN_OR_REQUIREMENTS]`——它本该做什么（计划文件路径、任务原文、或需求）
- `[BASE_SHA]`——起始提交
- `[HEAD_SHA]`——结束提交

**3. 对反馈按四类处理：**

| 分类 | 处置 |
|---|---|
| **Critical** | **立即修**，不许攒着 |
| **Important** | **继续往下做之前修掉** |
| **Minor** | 记下来（进进度 ledger），不当场修 |
| **reviewer 判断错了** | **带技术理由反驳**，不是沉默照做 |

## Example

```
[刚完成 Task 2：加验证函数]

You: 开工前先请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发代码审查子 Agent]
  DESCRIPTION: 新增 verifyIndex() 与 repairIndex()，覆盖 4 种 issue 类型
  PLAN_OR_REQUIREMENTS: 计划里的 Task 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[子 Agent 返回]:
  Strengths: 架构干净，测试是真的
  Issues:
    Important: 缺进度提示
    Minor: 上报间隔用了魔数（100）
  Assessment: Ready to proceed

You: [修掉进度提示]
[继续 Task 3]
```

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll just review the diff myself instead of dispatching a reviewer" | You're the coordinator — reviewing the diff inline burns the context window you need to keep driving the work. Dispatch a reviewer subagent: the diff and the evaluation live in its context, and only the findings come back to you. |
| "The reviewer needs my whole session history to understand the change" | Hand it precisely crafted context, never your session's history. That keeps the reviewer on the work product, not your thought process. |

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 收到了 review 反馈、准备动手改 | `receiving-code-review`——**先验后改**，不要盲目服从 |
| 每个任务后做任务级审查 | `subagent-driven-development`——本 skill 是其任务循环里的那道门 |
| 终局全分支宽范围审查 | 本 skill + [references/code-reviewer.md](references/code-reviewer.md)，用最强档模型派发 |
| 审查者说"测试不够" | `test-driven-development`（含 `references/writing-good-tests.md` 的测试质量判据） |
| 审查者指出实现过度、抽象多余 | `ponytail` / `ponytail-review`——复杂度审计由那两把尺子量刑 |
| 审查者指出根因不明、反复修不好 | `systematic-debugging`——先查根因再改 |
| 准备声称"已经修好了" | `verification-before-completion`——**先验证再声明**，不许凭"我改完了"结案 |
| 审查通过、准备合并 | `finishing-a-development-branch` |

**本 skill 管"什么时候派人审、怎么派人审"；它不管"收到意见之后怎么办"——那是 `receiving-code-review`。**

**模板见：** [references/code-reviewer.md](references/code-reviewer.md)
