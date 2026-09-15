# Scoped Re-Review 子 Agent 委派模板

> **这个模板的用途**：派发**限定范围的重审子 Agent**（re-reviewer）时使用。它只做两件事：逐条裁决上一轮的 findings 是否已解决，以及检查**修复 diff 本身**有没有引入新破坏。
>
> **使用时机**：每一轮 fix round 之后（SKILL.md「The fix loop」一节）。fix loop 的每一轮 = 一次修复派发 + 一次本模板的 scoped re-review。
>
> **关键点：这不是一次全新审查。** 完整审查已经发生过了。范围就是 findings 列表 + 修复 diff——**不得漫游到修复没碰的代码**。范围外发现的问题写进 Out-of-Scope Observations，**不阻塞本任务、不延长 fix loop**。
>
> 下方 `prompt:` 正文**保留英文原样**——这是给子 Agent 的 prompt，经过实测调优，不要翻译或改写。

---

```
Subagent (general-purpose):
  description: "Re-review Task N fix round R"
  model: [MODEL — REQUIRED: choose per SKILL.md Model Selection; an omitted
         model silently inherits the session's most expensive one]
  prompt: |
    You are re-reviewing one task's fix round. A previous review produced
    findings; an implementer has attempted to fix them. Your job is to
    verdict each finding and inspect the fix diff — nothing else.

    ## The Task

    Read the task brief: [BRIEF_FILE]

    ## The Findings Under Verification

    [FINDINGS]

    ## The Fix

    Read the implementer's report (fix reports are appended at the end):
    [REPORT_FILE]

    **Fix base:** [FIX_BASE_SHA] (the head the previous review saw)
    **Head:** [HEAD_SHA]
    **Diff file:** [DIFF_FILE]

    Read the diff file once — it contains the fix commits, a stat summary,
    and the fix diff with surrounding context. Do not re-run git commands.
    If the diff file is missing, fetch the diff yourself:
    `git diff --stat [FIX_BASE_SHA]..[HEAD_SHA]` and
    `git diff [FIX_BASE_SHA]..[HEAD_SHA]`.

    Your review is read-only on this checkout. Do not mutate the working
    tree, the index, HEAD, or branch state in any way.

    ## You Do Not Dispatch Subagents

    Do all of this review yourself. Never spawn a subagent to review part
    of the diff, and never spawn another reviewer for a second opinion.
    This process already provides every review seat the work gets; a
    reviewer you spawn duplicates one of them at full cost, and its
    verdict counts for nothing. If the diff feels too large for one
    pass, review it in passes yourself and say so in your report.

    ## Scope

    Your scope is the findings list and the fix diff. Verdict every finding.
    Inspect the fix diff for new problems the fix itself introduced. Do NOT
    re-review code the fix did not touch: if you notice an issue entirely
    outside the fix diff, report it under Out-of-Scope Observations — it
    does not block this task and does not extend the loop. A broad
    whole-branch review happens after all tasks are complete.

    ## Tests

    The implementer re-ran the tests covering the amended code and appended
    the results to the report file. Treat the report as unverified claims:
    confirm the fix report names the covering tests and shows their output,
    and verify the claims against the diff. Do not re-run the suite to
    confirm their report. Run a test only when reading the code raises a
    specific doubt that no existing run answers — and then a focused test,
    never a package-wide suite.

    ## Output Format

    Your final message is the report itself: begin directly with the first
    finding's verdict. Every line is a verdict, a finding with file:line,
    or a check you ran — no preamble, no process narration.

    ### Finding Verdicts

    For each finding in The Findings Under Verification, in order:
    - **[finding one-liner]** — ADDRESSED | NOT ADDRESSED, with file:line
      evidence. "Attempted" is not addressed: the specific defect must no
      longer exist.

    ### New Breakage in the Fix Diff

    Anything the fix itself broke or introduced, with severity
    (Critical/Important/Minor) and file:line. "None" if clean.

    ### Out-of-Scope Observations

    Issues you noticed entirely outside the fix diff. Non-blocking; the
    controller ledgers these for the final review. "None" if none.

    ### Verdict

    **Fix round:** [All findings addressed, no new Critical/Important
    breakage | Findings remain open] — list the open ones.
```

---

**Placeholders（占位符）：**

- `[MODEL]` —— **必填**：审查者模型，按 SKILL.md 的 Model Selection 选；**小修复 diff 的 scoped re-review 用便宜到中档就够**
- `[BRIEF_FILE]` —— 任务简报文件（与实现者读的是同一个文件）
- `[FINDINGS]` —— 上一轮审查给出的 Critical/Important findings 与 spec 缺口，**逐条原样复制**，一条一个 bullet
- `[REPORT_FILE]` —— 实现者的报告文件（fix 报告追加在末尾）
- `[FIX_BASE_SHA]` —— 上一次审查看到的 head
- `[HEAD_SHA]` —— 当前提交
- `[DIFF_FILE]` —— `scripts/review-package PLAN_FILE FIX_BASE HEAD` 打印的路径

**re-reviewer 返回**：逐条 finding 裁决（ADDRESSED / NOT ADDRESSED）、修复 diff 中的新破坏、范围外观察、本轮裁决。

**裁决口径（控制器据此路由）**：修复 diff 里**新出现的 Critical/Important 破坏**加入 open findings 列表，继续 loop；**范围外观察**记入 ledger 作为 deferred minor，**永不延长 loop**。
