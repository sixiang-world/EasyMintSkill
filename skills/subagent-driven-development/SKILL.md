---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

用**每个任务派一个全新实现者子 Agent**、**每个任务后做一次任务审查（spec 合规 + 代码质量）**、**终局做一次全分支宽范围审查**的方式执行计划。

**Why subagents:** 你把任务委派给**上下文隔离**的专职 Agent。通过精确构造它们的指令与上下文，你保证它们聚焦、成功。它们**永远不应该继承你的会话上下文或历史**——你需要什么，就构造什么给它。这同时把你的上下文留给协调工作。

**Core principle:** 每任务全新子 Agent + 任务审查（spec + 质量）+ 终局宽范围审查 = 高质量、快迭代

**Narration:** 工具调用之间，最多叙述一行短句——ledger 和工具结果承载记录。

**Continuous execution:** **不要在任务之间停下来和用户确认。** 把计划里的任务全部执行完，中间不停。唯二能停的理由是下面点名的四种情况，或者全部任务完成。"要我继续吗？"这类询问和进度汇报都在浪费用户的时间——他们让你执行这份计划，那就执行。

**Rulings, not stalls.** 一份正在跑的计划不等人类。冲突、歧义、计划缺陷、你本来打算申请突破的上限——**你自己裁决**。spec 是有约束力的权威，计划是它论证出来的东西，你的判断决定前两者都没回答的事。每个决定都记进 ledger：

```
Ruling: <what you decided> — <why> — <what it costs if wrong>
```

然后继续走。**一个判断错的 ruling，代价是用户看得见、撤得回的返工；一个停在问题上等的会话，代价是他们一整天，而且什么也没换来。**

**四件事能让你停下，且只有这四件：**
- 不可逆或有破坏性的操作
- 安全敏感的动作
- 在本 worktree 之外产生副作用、按惯例得先问的动作（合并、推送到共享分支、发布）
- 计划坏到**每条前进路径都是猜**

遇到这四种，停下，问。

## When to Use

```
有实施计划？
├─ 没有 → 手工执行，或先回去做头脑风暴
└─ 有 → 任务基本互相独立？
        ├─ 否（紧耦合）→ 手工执行，或先回去做头脑风暴
        └─ 是 → 留在本会话？
                ├─ 是 → subagent-driven-development（本 skill）
                └─ 否（另开会话）→ executing-plans
```

**vs. Executing Plans（另开会话）：**
- 同一会话内（没有上下文切换）
- 每任务全新子 Agent（没有上下文污染）
- 每任务后审查（spec 合规 + 代码质量），终局宽范围审查
- 迭代更快（任务之间没有人参与循环）

## The Process

```
Setup: 隔离工作区, ledger 检查, 读计划, 预检扫描
  │
  └─► 【Per Task 循环】
      │
      ├─ 1. 记录 BASE (git rev-parse HEAD)
      ├─ 2. 生成 task brief → 派发实现者子 Agent (references/implementer-prompt.md)
      │       │
      │       └─ 实现者有疑问？
      │            ├─ 是 → 回答问题、补充上下文 → 回到实现
      │            └─ 否 → 实现
      │
      ├─ 3. 实现者实现 / 测试 / 提交 / 自查 → 回传四种状态之一
      │       ├─ DONE                → 走审查
      │       ├─ DONE_WITH_CONCERNS  → 读顾虑，判是否先处理 → 走审查
      │       ├─ NEEDS_CONTEXT       → 补上下文，重新派发
      │       └─ BLOCKED             → 四路分流（见 Step 2）
      │
      ├─ 4. 生成 review package → 派发任务审查者 (references/task-reviewer-prompt.md)
      │       │
      │       └─ spec ✅ 且 质量 Approved？
      │            ├─ 是 → 标记完成，进 ledger
      │            └─ 否 → 走 fix loop
      │
      ├─ 5. Fix loop（每轮 = 一次修复派发 + 一次 scoped re-review）
      │       │
      │       ├─ finding 与计划文本冲突？
      │       │    └─ 是 → 先裁决冲突、记 ledger → 然后修
      │       ├─ 第 R 轮（R ≤ 3）：续用原实现者（带上 findings 原文）
      │       ├─ 第 R 轮（R ≥ 4）：换全新实现者 + 至少高一档的模型
      │       ├─ 派发 scoped re-review (references/re-review-prompt.md)
      │       │    └─ 全部 finding 已解决？
      │       │         ├─ 是 → 标记完成，进 ledger
      │       │         └─ 否 → R = 5？
      │       │              ├─ 否 → 下一轮
      │       │              └─ 是 → breaker 触发 ↓
      │       └─ Breaker（第 5 轮仍有 open finding）：逐条 adjudicate
      │            ├─ 有 load-bearing finding？
      │            │    ├─ 是 → 裁决最小解封改动 + 记 ledger + 带入下个任务派发
      │            │    │        （仅当每条前进路径都是猜时才停）
      │            │    └─ 否 → park + 记 ruling
      │            └───────────────────────────────────┘
      │
      ├─ 6. 把完成行追加进 ledger，标记任务清单项完成
      └─ 还有任务？
           ├─ 是 → 回到 1
           └─ 否 ↓
              │
              └─► 终局全分支审查：用最强模型派发 code reviewer
                    （`requesting-code-review` + 其 `references/code-reviewer.md`）
                    │
                    ├─ 有 findings？
                    │    └─ 是 → 【一次】修复派发 + 【一次】scoped re-review + adjudicate 残余
                    │             （没有第二轮修复波）
                    └─ 审查干净 → 删除本计划的 workspace
                                    │
                                    └─► 用 finishing-a-development-branch 收尾
```

## Setup

确保工作在**隔离工作区**里进行：用 `using-git-worktrees` 创建，或核实已有工作区。**没有用户明确同意，绝不在 main/master 分支上开始实现。**

会话记忆活不过上下文压缩。真实会话里，迷失位置的控制器曾把**整段已完成的任务序列重新派发一遍**——这是观测到的最昂贵的单一失败。**进度记在 ledger 文件里，不要只记在任务清单里。**

- 每份计划拥有一个 workspace：skill 启动时切到本 skill 的目录，跑
  `scripts/sdd-workspace PLAN_FILE` ——它会打印本计划的 git-ignored 目录
  （`<repo-root>/.agentskill/sdd/<plan-basename>/`），本计划的**全部**工件都住在这里：ledger、brief、report、review package。
  **另一份计划的目录，永远不该由你读写。**
- 在 `<workspace>/progress.md` 找本计划的 ledger。如果它的**第一行点的是你的计划文件**，那么带 `Task <N>: complete` 行的任务就是 DONE ——**不要重新派发**，从第一个没有该行的任务继续。
  某个任务最后一行是 fix round，说明它正处在 loop 中：从下一轮接着跑。
  **第一行点的是另一份计划文件**的 ledger——或者残留在旧扁平路径 `.agentskill/sdd/progress.md` 的 ledger——那是**别的**计划的进度：原地不动，自己新建一份。
- 新建 ledger，第一行就是它的身份：
  `# SDD ledger — plan: <plan file path>`
- ledger 是你的**恢复地图**：它点名的提交在 git 里真实存在，哪怕你的上下文已经忘了自己创建过它们。压缩之后，**信 ledger 和 `git log`，不要信自己的记忆。**
- `git clean -fdx` 会毁掉 workspace（它是 git-ignored 的临时区）；真发生了，从 `git log` 恢复。

把计划读一遍，记下它的上下文和 Global Constraints，为每个任务建一条任务清单项。**如果计划点名了 Spec，也读那份 spec**：spec 是计划论证所依据的权威，计划内部的冲突按 spec 裁决。**拿不到 spec 的计划，在 ledger 里记一句说明**——没有 spec 时做出的 ruling 都是临时性的。

在派发 Task 1 之前，把计划扫一遍找冲突，**边查边写下你查了什么**：

- 任务之间互相矛盾，或与计划的 Global Constraints 矛盾
- 计划明确要求、但审查准则视为缺陷的东西（一个什么都没断言的测试、一整块逻辑的逐字复制）

**扫描的产出是一张表，不是一个结论。** 每一对共享文件或接口的任务占一行：两个任务分别是什么、一个产出什么而另一个消费什么、你发现了什么。每个任务各占一行：它的文本**自己跟自己**是否一致——它规定的测试与它规定的代码、它创建的文件与它后来要动的文件。**没有这些行的"扫描是干净的"，是一句你没跑过的扫描。**

把这张表写进 ledger。**在执行开始之前**对发现的每一条做出裁决——每条都对着要求它的计划文本裁——并把每条 ruling 记进 ledger。扫描干净就直接走，不用评论。裁决它查出的每个冲突——spec 是有约束力的权威，计划是它的论证——把 ruling 记在对应那行旁边，然后派发 Task 1。**审查 loop 仍是兜住"只有实现才会暴露的冲突"的那张网。**

## Model Selection

用**能胜任该角色的最弱模型**，以省成本、提速度。

**机械实现任务**（孤立的函数、清晰的 spec、1-2 个文件）：用快而便宜的模型。计划写得好时，大多数实现任务都是机械的。

**集成与判断任务**（多文件协调、模式匹配、调试）：用标准档模型。

**架构与设计任务**：用可用的最强档模型。**终局全分支审查就是这一类**——用可用的最强档模型派发，不要用会话默认模型。

**审查任务**：用同一套判断标准选模型，按 diff 的**规模、复杂度、风险**缩放。小的机械 diff 不需要最强模型；微妙的并发改动需要。小修复 diff 的 scoped re-review 用便宜到中档。

**Fix-loop 升级（第 4-5 轮）**：用**至少比卡住的实现者高一档**的模型。

**派发子 Agent 时永远显式指定模型/能力档位。** 省略模型会继承你的会话模型——往往是能力最强也最贵的那一档——这会静默地废掉本节的全部意图。

**轮数胜过 token 单价。** 墙钟时间和上下文成本随子 Agent 的**轮数**增长，而最便宜的模型在多步工作上经常花 2-3 倍的轮数——总体更贵。**审查者**和**看散文式描述干活的实现者**，用中档模型当地板。当任务的计划文本里含有要写的完整代码时，实现就是"誊写 + 测试"：那种实现者用最便宜档。单文件机械修复也用最便宜档。

**任务复杂度信号（实现任务）：**
- 碰 1-2 个文件、spec 完整 → 便宜档
- 碰多个文件、有集成顾虑 → 标准档
- 需要设计判断或对代码库的广义理解 → 最强档

## The Task Loop

**批量打包同形小活。** 当计划里有若干个任务，各自都是**同一种、小而独立的编辑**——同一个一行修复、常量变更或字段添加在多个文件里重复——**不要一个任务派一个子 Agent**。把每个文件及其改动写进**一份**派发简报，整批发给一个子 Agent，把它的 diff 当作一个单元来审查。**"一任务一派发"留给那些需要自己的判断、自己的测试、或自己的审查面的工作。**

你粘进派发 prompt 的一切——以及子 Agent 打印回来的一切——**都会在你的上下文里驻留到会话结束，并在之后每一轮被重新读取**。**工件用文件交接。**

**等待已派发的子 Agent：** 永远不要用短超时轮询等待接口，也不要坐在一次沉默的、无期限的等待里。手上有本地活时（更新 ledger、打包下一份 review、读报告）就继续干；子结果自己会到。**真的空闲时，把等待切成有界的段**（五分钟到十分钟，在你平台允许的范围内），每段之间发一行状态并清点活着的子 Agent：列出来，追一下那些已经结束却没回报的。有界等待能保住长等待几乎全部的效率，同时保证卡住或丢失的子 Agent 在**几分钟内**被发现，而不是在会话末尾。

### 1. Dispatch the implementer

**派发前记录 BASE（`git rev-parse HEAD`）**——review package 和 fix-round diff 都要用它。

- **Task brief:** 派发实现者之前，切到本 skill 目录跑 `scripts/task-brief PLAN_FILE N`——它把该任务的完整文本抽到一个唯一命名的文件，并打印路径。**派发内容要让 brief 保持为需求描述的单一来源。**
  你的派发应当包含**五项**：
  1. 一行说明这个任务在项目里的位置
  2. brief 路径，并以"先读这个——它是你的需求，里面是原样照用的精确值"引出
  3. 前面任务留下的、**brief 不可能知道**的接口与决定
  4. 你对 brief 中注意到的任何歧义的处置
  5. 报告文件路径与报告契约

  **精确值（数字、magic string、签名、测试用例）只出现在 brief 里。** **Never make a subagent read the whole plan file.**
- **Report file:** 实现者的报告文件按 brief 命名（brief `…/task-N-brief.md` → report `…/task-N-report.md`），并把路径放进派发 prompt。实现者把完整报告写在那里，**回传时只回状态、提交、一行测试摘要和顾虑**。
- **一份派发 prompt 描述的是"一个任务"，不是"这个会话的历史"。** **不要把累积的前序任务摘要（"Tasks 1-3 之后的状态"）粘进后面的派发**——一个真实会话的派发 prompt 达到了 **42k 字符，其中 99% 是粘贴的历史**。一个全新的子 Agent 需要的是：它的任务、它要碰的接口、全局约束。**别的一概不需要。**
- 派发内容携带 **no-subagents contract**（它就在实现者模板里）：实现者**永不派发子 Agent**——不派帮手，**尤其不派审查者**。审查由你**在报告之后**送过去。真实会话里，工人自己派出的每一个审查者，都在**重复控制器本来就会派发的那次任务审查**——每个任务白多出一个审查座席。
- 如果更早的某个任务在**本任务要碰的区域**park 过 finding，派发里带上那条 ledger 记录的指针。
- 从派发结果里**记下实现者的 agent 身份**——fix loop 第 1-3 轮要续用它。
- **永不并行派发多个实现子 Agent**（会冲突）。

模板：[references/implementer-prompt.md](references/implementer-prompt.md)

### 2. Handle the report

实现者子 Agent 会回报四种状态之一。分别处理：

**DONE：** 生成 review package（切到本 skill 目录跑 `scripts/review-package PLAN_FILE BASE HEAD`——它打印自己写入的唯一文件路径；**BASE 是你派发实现者前记录的那个提交**——**绝不要用 `HEAD~1`，它会静默丢掉多提交任务里除最后一个之外的所有提交**），然后用打印出的路径派发任务审查者。

**DONE_WITH_CONCERNS：** 实现者完成了工作，但标了疑虑。**先读顾虑再往下走。** 如果顾虑关乎正确性或范围，**先解决再审查**。如果只是观察（例如"这个文件越来越大了"），记一句，继续走审查。

**NEEDS_CONTEXT：** 实现者需要没被提供的信息。把缺的上下文补上，重新派发。

**BLOCKED：** 实现者无法完成该任务。评估阻塞：
1. 如果是**上下文问题** → 补充更多上下文，**用同一个模型**重新派发
2. 如果任务**需要更强的推理** → 换**更强档的模型**重新派发
3. 如果任务**太大** → 拆成更小的块
4. 如果**计划本身是错的** → 裁决这个修正、记进 ledger，并把 ruling 带在派发内容里重新派发

**绝不**忽略一次升级，也**绝不**在什么都不改的情况下逼同一个模型重试。**实现者说自己卡住了，那就必须有什么东西发生变化。**

如果实现者提问——开工前或干活中途——**清楚完整地回答**，需要时补充上下文，不要催它赶紧进入实现。

### 3. Review the task

**每任务审查是任务级的门。** 宽范围审查只有一次，在终局全分支审查处。**永不跳过任务审查**，也**永不接受缺少任一裁决的报告**——spec 合规**和**任务质量两者都是必需的。**实现者的自查永不替代任务审查；两个都要。**

**一个审查者，读一次 diff，返回两个裁决。** 它先判 spec 合规（Part 1），再判代码质量（Part 2）。**这不是两个子 Agent**——两个审查者=两倍成本，而一个审查者一次读 diff 就足以给出两个裁决。审查者的完整输出格式见下面的模板。

- **把 diff 作为文件交给审查者：** 跑本 skill 的
  `scripts/review-package PLAN_FILE BASE HEAD`，把打印出的文件路径给审查者
  （没有 bash 时：把 `git log --oneline`、`git diff --stat`、`git diff -U10` 这段范围的输出重定向到一个唯一命名的文件）。
  **输出永远不进你自己的上下文**，而审查者一次 Read 就能看到提交列表、stat 摘要和带上下文的完整 diff。**用你派发实现者前记录的那个 BASE——绝不要用 `HEAD~1`，它会静默截断多提交任务。** **绝不在没有 diff 文件的情况下派发任务审查者。**
- **审查者的输入：** 任务审查者拿到三个路径——**同一个** brief 文件、report 文件、review package——**外加**约束该任务的全局约束。
- 你交给审查者的 **global-constraints block 就是它的注意力透镜**。从计划的 Global Constraints 节或 spec **原样复制**有约束力的要求：精确值、精确格式、组件之间声明的关联关系（"与 X 同一布局"、"与 Y 相同"）。审查者模板里已经带了流程规则（YAGNI、测试卫生、审查方法）——约束 block 负责的是**本项目的 spec 要求什么**。
- **不要**在没有具体、任务相关的理由时加"检查所有用法"或"如果合适就跑竞态测试"这类开放式指令
- **不要**让审查者重跑实现者在同一份代码上已经跑过的测试——实现者的报告里带着测试证据
- **绝不为审查者预判 findings**——**绝不指示审查者忽略或不报某个具体问题**。如果你认为某条 finding 会是误报，**让审查者提出来，在 review loop 里裁决它**。如果你正在写的 prompt 里出现了 **"do not flag"、"don't treat X as a defect"、"at most Minor"、"the plan chose"**——**停下：你正在预判，通常是为了给自己省掉一轮 review loop。**

任务审查者可能报出 **"⚠️ Cannot verify from diff"** 条目——那些要求存在于未改动的代码里，或跨任务。**它们不阻断审查的其余部分，但你必须在把任务标记为完成之前，自己逐条解决**：**你**手上才有审查者缺的那份计划和跨任务上下文。如果你确认某条是真实缺口，**按 spec review 失败处理**——它和其他 findings 一起**进 fix loop**。

模板：[references/task-reviewer-prompt.md](references/task-reviewer-prompt.md)

### 4. The fix loop

**触发条件**：审查报出 spec ❌、任何 Critical 或 Important finding、或你确认为真实缺口的 ⚠️ 条目。

**在 loop 开始前，有两条路立刻离开 loop：**

- **Minor findings 边做边记进进度 ledger**（`Task <N>: minor (deferred): <one-liner>`），**并把终局全分支审查指向这份清单**，让它来分诊哪些必须在合并前修掉。**一份没人读的汇总就是静默丢弃。Minor findings 永不进 loop。**
- **被标为 plan-mandated 的 finding——或任何与计划文本要求冲突的 finding——由你来裁决**：把它放上秤、对着计划文本称，以 spec 为有约束力的权威做决定，**动手之前先把 ruling 记进 ledger**。**不要因为"计划要求这么做"就把 finding 驳掉，也不要没有记录的 ruling 就派发一个与计划相悖的修复。**

其余一切进 loop。**一轮 fix round = 一次修复派发 + 一次 scoped re-review。每个任务最多五轮。**

**第 1-3 轮——续用原实现者。** 把 open findings **原样**发给它。它的上下文还完好：它知道这个任务、这些代码、以及它自己的选择。**如果你的运行时不能再给一个活着的子 Agent 发消息**，就派一个全新实现者，带上 brief 路径、报告文件路径和 findings——**无论哪条路，报告文件都是那份持续存在的记忆。**

**第 4-5 轮——用更强档的模型派发全新实现者**（按 Model Selection），带上 brief 路径、报告文件路径、open findings，以及这段定调的话："A prior implementer attempted this task [N] times; you own it now. Read the report file for what was tried."（一名先前的实现者尝试过这个任务 [N] 次；现在归你。读报告文件看它试过什么。）**一个熬过三次续用的 loop，通常意味着实现者看不见自己的问题——新眼睛和能力上调一步到位。**

**每一轮都一样：** 实现者修复、重跑覆盖被改代码的测试、把 fix 报告**追加到同一个报告文件**、回传短契约。**重新派发审查者之前，确认 fix 报告里有覆盖测试、跑的命令和输出；三样齐了再派 re-review。** 在 fix 消息里**点名覆盖测试文件**——一个一行修复不需要整个套件。

**re-review 是限定范围的。** 跑 `scripts/review-package PLAN_FILE FIX_BASE HEAD`，其中 **FIX_BASE 是上一次审查看到的 head**，并用 findings 列表、brief、报告文件、打印出的 diff 路径派发 [references/re-review-prompt.md](references/re-review-prompt.md)。re-reviewer 对每条 finding 裁决 ADDRESSED 或 NOT ADDRESSED，并且**只在修复 diff 范围内**标记新破坏。**修复 diff 中新出现的 Critical/Important 破坏加入 open findings 列表。范围外的观察作为 deferred minors 进 ledger——它们永不延长 loop。**

**每轮之后**，往 ledger 追加：
```
Task <N>: fix round <R>/5 (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)
```

**绝不在控制器会话里自己修 findings**——你的上下文要保持干净用于协调，而且**控制器自己动手的修复会跳过审查**。

**The breaker.** 当第 5 轮的 re-review 仍有 finding open，**停止派发**。你自己逐条裁决 open finding——**你手上才有审查者缺的那份计划和跨任务上下文**：

- **审查者错了，或这一点可争议：** park 它——
  `Task <N>: parked — <finding> — Ruling: <why the code stands>`
  终局审查会看到双方。
- **是真的，但下游没有任何东西建立在它之上：** 同样 park，格式一样，但 ruling 要写明它**真实存在但延后处理**（park-as-real-deferred）——不是"代码站得住"，而是"问题是真的，但此刻不值得再花一轮"。
- **是真的且 load-bearing**——后续任务建立在它之上，或它暴露了一个计划缺陷：**裁决一个能解封依赖工作的最小改动**，记为
  `Task <N>: Ruling: <finding> — <what you decided and why>`，
  并把它带进下一个任务的派发。**静默地 park 一个结构性失败，会让每一个依赖它的任务都建在它上面。** **只有当缺陷使得每条前进路径都是猜的时候才停。**

**只在 cap 处裁决。** 提前裁决来终止 loop，是**换了个名字的预判**。每一次裁决都是一条 ledger 记录——**静默丢弃是禁止的。**

### 5. Complete the task

**当审查干净地回来**——或在 cap 处每条 open finding 都带 ruling 被 park 了——把完成行追加进 ledger，**和你其他的记账放在同一条消息里**：

- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <K> parked)`（breaker 触发之后）

然后标记任务清单项完成，继续。**只要审查里还挂着既没修、也没在 cap 处带着 ruling 被 park 的 Critical/Important 问题，就绝不进入下一个任务。**

## Final Review

**终局全分支审查也拿一份 package**：跑
`scripts/review-package PLAN_FILE MERGE_BASE HEAD`（**MERGE_BASE = 分支起点的提交**，例如 `git merge-base main HEAD`），把打印出的路径放进终局审查的派发里，**这样终局审查者读一个文件，而不是用 git 命令重新导出分支 diff**。用**可用的最强档模型**派发（见 Model Selection），并走 `requesting-code-review`（用其 `references/code-reviewer.md` 模板）。**把它指向 ledger 里 deferred-minor 和 parked 那些行**，让它分诊哪些必须在合并前修掉。

如果终局全分支审查返回 findings，**派发一次（ONE）修复子 Agent，带完整 findings 列表——不是一个 finding 一个修复者**。按 finding 分开的修复者各自重建上下文、各自重跑套件；**一个真实会话的终局审查修复波，比它所有任务加起来还贵**。然后**只做一次**修复波的 scoped re-review（对修复范围跑 `scripts/review-package PLAN_FILE FIX_BASE HEAD`，用 [references/re-review-prompt.md](references/re-review-prompt.md)）。剩余 findings 按任务 loop 的 breaker 同样裁决：park 并给 ruling，或对 load-bearing 的下裁决并记下你决定了什么。**这里也只有上面那四类事能让你停下。没有第二轮修复波**——残余的 load-bearing finding 在 `finishing-a-development-branch` 摆出选项时，浮现给用户。

## Finish

**在删任何东西之前**，把 ledger 里每一条含 `Ruling:` 的行——预检裁决、parked findings、breaker 裁决，**全部**——按你做出它们的顺序收进最终消息的 "Rulings I made" 之下，每条都写清错了要付什么代价。**这份清单是穷举的：ledger 里有的 ruling，清单里就有。** 这份清单是你替用户做的决定**唯一**能到达他们的地方——他们读了，然后把做错的重做。**A ruling that dies with the workspace was a decision made in secret.**

当终局全分支审查干净、其修复已合并，**删掉本计划的 workspace**（`rm -rf <workspace>`）——记录现在活在 git 历史里。**兄弟目录属于别的计划；别碰。**

用 `finishing-a-development-branch` 收尾。

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found spec gaps = not done. Fix or hit the cap and adjudicate — those are the only exits. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute your context and skip review. Resume the implementer. |
| "One more round will converge" | Past the cap, rounds don't converge — the failure is structural. Adjudicate and route. |
| "The reviewer will just find something new anyway" | Scoped re-reviews verify fixes; they cannot wander. New findings on untouched code go to the ledger, not the loop. |
| "This finding is obviously wrong, I'll drop it" | You adjudicate only at the cap, and every ruling is a ledger entry. Silent discards are forbidden. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. Every round ends with a scoped re-review. |
| "Reviews slow the loop down" | The loop without reviews is just unverified churn. Reviews are the loop's brakes and steering. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. Controllers without one have re-dispatched entire completed task sequences. |
| "The implementer spawned its own reviewer — free extra assurance" | It's a duplicate seat reviewing the same diff; the task review is the gate. A worker-spawned reviewer is a defect to flag, not rigor. |

## 主 Agent 的职责边界

控制器（主 Agent）的职责是**协调**，不是实现。这条边界是整套机制的成本前提：控制器一旦下场写代码，审查门就形同虚设。

| 该做 | 不该做 |
|---|---|
| 记录 BASE、生成 brief / report 路径、生成 review package | **Never fix findings yourself in the controller session**——修完的代码跳过了审查 |
| 派发实现者 / 任务审查者 / re-reviewer，并**显式指定模型档位** | 自己读 diff 并下质量结论（那是审查者的裁决） |
| 回答实现者的问题、补上下文、按四路分流处理 BLOCKED | **禁止预判 findings**——不指示审查者忽略某问题，不写 "do not flag" / "at most Minor" / "the plan chose" |
| 维护 ledger（完成行、fix round 行、minor 行、ruling 行） | 自己跑测试来替代实现者的测试证据 |
| 解决 ⚠️ Cannot verify from diff 条目（**必须自己解决，不得跳过**） | 并行派发多个实现子 Agent |
| 在 cap 处逐条 adjudicate、写 ruling、把 ruling 带进下游派发 | 在 cap 之前 adjudicate 以提前结束 loop（那是预判） |
| 终局时把全部 ruling 穷举回报给用户 | 让 ruling 随 workspace 一起被删掉 |

## Ledger 记账格式

**四种行格式，照抄：**

```
Task <N>: complete (commits <base7>..<head7>, review clean)
Task <N>: complete (commits <base7>..<head7>, <K> parked)
Task <N>: minor (deferred): <one-liner>
Task <N>: fix round <R>/5 (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)
```

park 与裁决的行：
```
Task <N>: parked — <finding> — Ruling: <why the code stands>
Task <N>: Ruling: <finding> — <what you decided and why>
```

预检扫描：**一张表**（见 Setup）——每对共享文件/接口的任务一行，每个任务一行自洽性检查——写进 ledger，ruling 记在对应行旁。

**ledger 第一行永远是**：`# SDD ledger — plan: <plan file path>`。**这一行是身份**：它决定这条 ledger 是不是你的。

## Example Workflow

```
You: I'm using Subagent-Driven Development to execute this plan.

[Setup: 隔离工作区已核实]
[只读一次计划文件: docs/plans/feature-plan.md]
[解析 workspace: scripts/sdd-workspace docs/plans/feature-plan.md — 里面没有 ledger，全新开始]
[为所有任务建任务清单项]

Task 1: Hook installation script

[为 Task 1 跑 task-brief；带 brief + report 路径 + 上下文派发实现者]

Implementer: "Before I begin - should the hook be installed at user or system level?"

You: "User level (~/.config/hooks/)"

Implementer: [稍后]
  - Implemented install-hook command
  - Added tests, 5/5 passing
  - Self-review: Found I missed --force flag, added it
  - Committed

[跑 review-package PLAN_FILE BASE HEAD；用打印出的路径派发任务审查者]
Task reviewer: Spec ✅ - all requirements met, nothing extra.
  Strengths: Good test coverage, clean. Issues: None. Task quality: Approved.

[Ledger: Task 1: complete (commits a1b2c3d..d4e5f6a, review clean)]

Task 2: Recovery modes

[为 Task 2 跑 task-brief；带 brief + report 路径 + 上下文派发实现者]

Implementer: [没有提问]
  - Added verify/repair modes
  - 8/8 tests passing
  - Committed

[跑 review-package PLAN_FILE BASE HEAD；用打印出的路径派发任务审查者]
Task reviewer: Spec ❌:
  - Missing: Progress reporting (spec says "report every 100 items")
  Issues (Important): Magic number (100)

[Fix round 1：带两条 findings 续用原实现者]
Implementer: Added progress reporting, extracted PROGRESS_INTERVAL constant.
  Re-ran test/recovery.test.js — 10/10 passing. Fix report appended.

[跑 review-package PLAN_FILE FIX_BASE HEAD；派发 scoped re-review]
Re-reviewer: Missing progress reporting — ADDRESSED (src/recovery.js:41).
  Magic number — ADDRESSED (src/recovery.js:7). New breakage: none.
  Verdict: all findings addressed.

[Ledger: Task 2: fix round 1/5 (2 addressed, 0 open; commits d4e5f6a..b7c8d9e)]
[Ledger: Task 2: complete (commits d4e5f6a..b7c8d9e, review clean)]

...

[全部任务之后]
[跑 review-package PLAN_FILE MERGE_BASE HEAD；用最强模型派发终局 code reviewer]
Final reviewer: All requirements met. Deferred minors triaged: none block merge.

[删掉本计划的 workspace —— 记录现在活在 git 里]

Done! Using finishing-a-development-branch.
```

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 还没有计划 | `writing-plans`——本 skill 的输入就是一份计划文档；没有 spec 则更早一步回创建期流程 |
| 任务之间**紧耦合**，切不开 | **不要用本 skill**——转 `executing-plans`，或先回去把计划重切 |
| 没有子 Agent 委派机制可用 | 转 `executing-plans` 并**显式说明这是降级**，不要假装有子 Agent |
| 还没建隔离工作区 | `using-git-worktrees`（Setup 第一步） |
| 实现者写代码的纪律 | `test-driven-development`——计划里 TDD 是默认要求 |
| 实现者中途撞上 bug、根因不明 | `systematic-debugging`——不要让它在本 loop 里硬猜着改 |
| 某个任务实现过度、冒出多余抽象 | `ponytail` / `ponytail-review`——审查者的 YAGNI 判断来源 |
| 终局全分支审查 | `requesting-code-review`（+ 其 `references/code-reviewer.md` 模板）；终局审查用最强档模型 |
| 需要独立问题域并发派发 | `dispatching-parallel-agents` |
| 收到了审查者的 findings、准备动手改 | `receiving-code-review`——先验后改，不盲目服从 |
| 终局审查干净、准备合并 | `finishing-a-development-branch`——由它摆出集成选项 |
| 准备声称某个任务完成 | `verification-before-completion` |

**本 skill 管"怎么跑任务循环"；它不产出计划、不合并分支、不替代 TDD——那些都有各自的 skill。**
