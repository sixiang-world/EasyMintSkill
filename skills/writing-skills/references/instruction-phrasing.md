# 负面指令的生效边界（实测版）

> **来源**：superpowers 项目（MIT License）设计文档
> `docs/superpowers/specs/2026-06-10-positive-instruction-redesign-design.md`，
> 已中立化整理（移除具体产品名与模型名），保留全部英文术语与原文关键句以便引用。
> 原文档状态为 **Proposed**；其中 writing-plans 部分另有后续实测结论，见下文「六、后续实测」。
>
> 本文回答一个 `writing-skills` 全篇在用、但从未说明的问题：
> **禁令（prohibition）什么情况下有效，什么情况下反而有害。**

---

## 一、核心发现（The measured finding）

对一个 controller 的措辞做**微测（micro-test）**，测量措辞如何改变 controller **组装出来的产出（composed artifact）**：

| Case | Phrasing | Result |
|---|---|---|
| Dispatch composition（"don't restate the brief"） | prohibition | **4.4** spec values re-typed — *worse than no guidance*（比不写指导还差，对照组 3.6） |
| Dispatch composition | positive recipe（"your dispatch should contain: (1)…(5)"） | **3.0, zero variance** —— 采纳 |
| Dispatch composition | recipe + nuance clause（"quote only the fragment…"） | **3.8, noisy** —— nuance 稀释配方 |
| Test-rerun directive（"do not ask reviewer to re-run tests"） | prohibition | **0/5 violations** —— 正常工作（对照组 3/5） |
| Test-rerun directive | positive recipe | 0/5 —— 等效，但更长 |

**一句话结论**：同样是禁令，有的零违规（tripwire / recognition table / discrete directive），
有的**比不写还差**（composition prohibition），而且这个差别**可预测**。

---

## 二、判据本体：五类指令的生效规律

原文称之为 **The doctrine**（"use this to classify any negative instruction"）：

1. **Tripwires work.** Phrase-level self-checks on concrete tokens（对具体 token 的措辞级自检）
   可靠触发 —— "if the prompt you are writing contains 'do not flag' … stop"。
2. **Recognition tables work.** Red-Flags / rationalization tables 在**决策时**（decision time）被读到，
   而不是在**组装时**（composition time）—— 因此不受组装期竞争性激励影响。
3. **Discrete-directive prohibitions work.** "Do not ask X to do Y" 成立的条件是：
   **模型对 "do Y" 没有竞争性激励**（no competing incentive）。
4. **Composition prohibitions backfire**（组装型禁令会反噬）—— 当模型对产出**有自己的议程**
   （its own agenda for the output）时，例如 "restating specs feels like helpful curation"（复述 spec
   感觉像是在做有用的信息组织）。**只有正向组装配方（positive composition recipe）能撬动这一类**；
   而且**给一个已经赢了的配方追加 nuance 从句只会让它更差，不会更好**。
5. **Ties go to the shorter phrasing.**（平局选更短的措辞。）长会话中 SKILL.md 会被反复重读
   （measured 2026-06-10：约 500× per long session），散文长度是**真实成本**。

### 分类判据表

| 类别 | 定义 | 生效判据 | 结论 |
|---|---|---|---|
| **Tripwire** | 对具体 token 的措辞级自检："如果你正在写的 prompt 里出现 'xxx'，停下" | 挂在可观测的字面 token 上，自检在**动笔时**即刻可判 | **保留** |
| **Recognition table** | Red-Flags / rationalization 表，列出失败形态与借口 | 读取时机是**决策时**，不是组装时 | **保留** |
| **Policy gate** | 跨产出的治理性规则（如 "never push without permission"） | 属于 policy，不是 composition shaping；不塑造产出形状 | **保留** |
| **Discrete-directive prohibition** | "Do not ask X to do Y" 型的具体动作禁令 | 模型对 "do Y" **没有竞争性激励** | **保留**（实测 0/5 违规） |
| **Composition prohibition** | "不要用 X 方式做某事"、禁止性清单，针对**产出本身该怎么写** | 模型对产出**有自己的议程**时——"少写/复述"看起来像在帮忙 | **必须改写**（实测比无指导更差） |

**最重要的分界线**：禁令是否在跟模型的**自身议程**对抗。
- 对抗"该不该做某个动作" → 通常有效。
- 对抗"产出该长什么样" → 通常反噬，必须换成 positive recipe。

---

## 三、审计结果（Audit results, 2026-06-10）

范围：全部约 **30 个 skill** + prompt 模板。计数：

- **3 个 tripwire**（keep）
- **14 个 recognition table**（keep）
- **约 20 个 policy gate**（keep —— "never push without permission" 是 policy，不是 composition shaping）
- **5 个 composition-prohibition**（逐个处置如下）

| # | Location | Disposition |
|---|---|---|
| 1 | `subagent-driven-development/task-reviewer-prompt.md` — "Cite, don't narrate" | **改**：以正向半边开头（"Your report should point at evidence: file:line for every finding…"），删掉禁令半边（dead weight —— 正向半边已经存在并且承担了全部负荷） |
| 2 | `subagent-driven-development/SKILL.md` — "Do not add open-ended directives" | **维持原样**：微测在 15 个样本中**无法诱发该失败**；双向都无证据；平局选短的 |
| 3 | `subagent-driven-development/SKILL.md` — "Do not ask a reviewer to re-run tests" | **维持原样**：实测 **0/5 违规**；且该禁令会**自我传播**进 dispatch 里，有额外价值 |
| 4 | `subagent-driven-development/SKILL.md` — "do not re-review on top of it" | **改**：换成三要素清单（"Before re-dispatching the reviewer, confirm the fix report contains: the covering tests, the command run, and the output"） |
| 5 | `writing-plans/SKILL.md` — the "No Placeholders" banned-patterns list | **边界案例**，见下节 |

另有 borderline 一条，随 #5 推迟：
`task-reviewer-prompt.md` — "Don't flag pre-existing file sizes — focus on what this change contributed"
（正向半边已存在且承担负荷；影响低）。

---

## 四、边界案例：为什么 "No Placeholders" 不能盲目改成正向清单

`writing-plans/SKILL.md` 的 "No Placeholders" 结构是：**一句正向陈述**
（"Every step must contain the actual content an engineer needs"）+ **六条 banned-patterns 清单**
（"never write them: 'TBD', 'TODO', 'Add appropriate error handling', 'Write tests for the above', 'Similar to Task N', …"）。

原文明确指出它**同时具备两种相反特征**，因此"不确定"是真实的：

- 计划是工作流中**最大的生成产物**，模型对产出有**真实的竞争性激励**去写 placeholder
  （在长度压力下，placeholder 是最省力路径）—— 这是"禁令实测反噬"那一类的**激励结构**。
- 但被禁的条目是**离散、可识别的 token** —— 这是"禁令实测有效"那一类的**形态**。
- **该清单还在别处承担负荷**：skill 的 Self-Review 段引用了它
  （"Placeholder scan: search your plan for red flags — any of the patterns from the 'No Placeholders' section above"）。
  这些 token 同时充当**复查期的扫描清单**，而 review-time recognition 恰好是**有效的那一类**。
  盲目换成正向 checklist 会**打断这个引用**，并丢掉好的 tripwire token。

### 重构方案（V2，by mechanism）

- **V0（现状）**：组装期 = 正向句 + banned list；Self-Review 引用该 list。
- **V1（auditor's checklist）**：组装期只留正向配方 ——
  "Before finalizing a step, confirm it has: the literal code to write, a runnable command with
  expected output, types and method names defined within this plan, error handling shown explicitly.
  A step is complete when an engineer could implement it without asking any follow-up questions."
  Self-Review 保留泛化的 placeholder 扫描。
- **V2（按机制重构，预测最优）**：组装期只拿 V1 的正向配方；
  **把具名 patterns 整体搬进 Self-Review 的 placeholder-scan 步骤，改写为 recognition 形态**
  （"when you scan, look for: 'TBD', 'TODO', 'Similar to Task N', …"）。
  **同样的 token，从"会诱导的那一类"搬到"能检测的那一类"**（relocated from the category that primes
  to the category that detects）。
- **V3（对照）**：只有正向句，任何地方都不放清单。

**V2 是本文最重要的可复用手法**：一条禁令若同时是**好 token** 和**坏位置**，
不要删掉它，而是**搬位置**——从组装期搬到复查期。

---

## 五、实验设计（Micro-test design，决定数据可信度）

### 微测 harness 方法

- **每个样本一次调用**：system prompt = 该 skill 指导语变体，放在**真实的周边上下文**里；
  user = 一个**真实的流程中段场景**；output = 组装出来的产物（dispatch prompt / plan / report）。
- **程序化打分**：用 grep 找无歧义标记；但**每一个命中都要人工复核再下结论**
  —— 当晚有一个所谓 "violation" 其实是 controller 在**正确地引用那条禁令**，
  另一个被自动化否定检测误判。
- **成本**：约 $0.15–0.30/sample，每次迭代**几秒**，对比完整评测 **$12 / 50 分钟**。
  措辞在这里迭代；只有当改动是**结构性**的，才用完整跑确认赢家。
- **永远带一个无指导对照组（no-guidance control）** —— 正是它揭示了
  一次**反噬**（复述：禁令比什么都不写更差）和一次**有效的禁令**
  （重跑测试：对照组 3/5 失败 vs 任一种措辞 0/5）。

### writing-plans 微测的具体设计

- **任务**：用一份**故意欠规格的 spec** 写 2–3 个任务的实施计划（欠规格才会诱发 placeholder）。
  fixture spec 含：一个规格良好的任务、一个 error handling 被含糊带过的任务、
  一个与第一个任务相似的任务（诱导 "Similar to Task 1"）。
- **采样**：**每个变体 5+ reps**，默认 temperature。
- **程序化打分**（除注明外，越低越好）：
  - banned-token count：
    `TBD|TODO|implement later|fill in details|appropriate error handling|handle edge cases|Similar to Task|Write tests for the above`
  - 该改代码却没有 fenced code block 的步骤数
  - 引用了计划中**任何地方都未定义**的类型/函数的次数
  - （越高越好）每个任务中**带预期输出的可运行命令**数
- **V2 的两段打分**：还要测 Self-Review 那一半 —— 把生成的计划连同该变体的 Self-Review 段喂回去，
  测扫描是否真能抓到**植入的 placeholder**（往 fixture plan 里插 2 个已知 placeholder，检测率为指标）。
- **采纳标准（Acceptance）**：只有当一个变体在 banned-token count 上**打败 V0**，
  且**不掉** code-block coverage 或 self-review detection rate，才采纳。预计总成本 ~$6–10。

---

## 六、后续实测（Result, run 2026-06-10）

**结论：无需改动（Resolved — no change needed）。**

- **Stage 1**（3 任务 spec，无压力）：**4 个变体 × 全部 20 份计划 = 0 个 placeholder**
  —— 包含**无指导对照组**在内。
- **Stage 1b**（10 任务 spec，五个近乎相同的命令诱导 "Similar to Task N"，
  外加明确的约 2,500 词篇幅目标）：**40/40 干净** —— 唯一的正则命中是
  V2 的 self-review 在**确认** "no TBD/TODO ✓"。

**处置**：No Placeholders 段**原样保留**（成本低，且反事实不可测）；**不要开后续 PR**。
V2 的搬位置设计**留档于此**，以备未来模型代际**回归（regress）**时启用。

### 原文明确排除、附数据的几条（"tested-and-declined, with data"）

记录在此，以免有人在**没有新证据**的情况下重新提议：

- **Controller turn batching / 单条消息并行工具调用**：controller **每条消息恰好发一个工具调用**
  （所有测量运行中 **0 条 multi-tool 消息**，有指导无指导都一样）。
  **46%** 的 controller turn 是思考/叙述、不含工具调用 —— 这是一个 **prompt-immune floor**（措辞无法撼动的下限）。
- **通过 `run_in_background` 做 pipelined reviews**：机制在提供时会被采纳（**7/28 dispatches**），
  但在 45 分钟场景下收益**低于 run-to-run 噪声底**（单次 review 仅约 30–60s）；还引入双结果流协调问题。
  只有当计划中的 review 单项耗时很长时才值得重看。
- **给获胜配方追加 nuance 从句**：**可测量地使其退化**
  （C2: 3.8 noisy vs C: 3.0 consistent）。
  **迭代方式是重新推导配方（re-deriving the recipe），而不是追加 caveat。**

---

## 七、实用指南：手上一句禁令，怎么判

按顺序走完这 6 步。

**Step 1 — 它是 policy 还是 composition shaping？**
是治理性规则（权限、安全、不可逆操作）→ **Policy gate，保留**，不参与本流程。
（原文判例："never push without permission" 是 policy。）

**Step 2 — 它挂在具体字面 token 上吗？**
是（如 "if the prompt contains 'do not flag' … stop"）→ **Tripwire，保留**。零改动。

**Step 3 — 它是在决策时被读、还是在组装时被读？**
它的自然位置是**列表/表格**，在判断"这是不是失败形态"时被查 → **Recognition table，保留**。
位置在"你现在要写的这份产出"旁边 → 继续 Step 4。

**Step 4 — 模型对 "do Y" 有没有竞争性激励？**
问自己：**照禁令做，模型是否有理由觉得"那是在帮忙"？**
- 没有（如 "不要要求 reviewer 重跑测试" —— 模型没有理由非要它重跑）→
  **Discrete-directive prohibition，保留**。平局时选**更短**的措辞（详见 Step 6）。
- 有（"不要复述 spec" —— 复述感觉像在提供有用信息）→
  **Composition prohibition，必须改写** → Step 5。

**Step 5 — 改写成 positive recipe（组装期）。**
给出**产出由哪些部分组成、按什么顺序**的清单：

```text
反例（会反噬）：
  "don't restate the brief"

正例（positive recipe）：
  "your dispatch should contain: (1)… (2)… (3)… (4)… (5)…"
  "Before finalizing a step, confirm it has: the literal code to write,
   a runnable command with expected output, types and method names defined
   within this plan, error handling shown explicitly."
```

改写时守两条铁律：
- **不要追加 nuance 从句。** 给已经赢了的配方加 "unless it matters" 之类，
  会**把谈判重新打开**，实测把结果从稳定（3.0）降到嘈杂（3.8）。
  真有例外 → 写成**挂在可观测谓词上的独立条件句**，或**重新推导配方**。
- **不要用豁免条款来划范围。** "This limit doesn't apply to code blocks" 依然会压制 code blocks；
  应**重构结构让规则够不到它**。

**Step 6 — 平局就选短的。** SKILL.md 会被反复重读，散文长度是真实成本。
正向配方与禁令都 0/5 违规时（如 test-rerun case），留**短的那个**。

### 特殊情况：它既是好 token，又处在坏位置

不要删。**搬位置**：把 token 从**组装期**搬到**决策/复查期**，改写成 recognition 形态
（"when you scan, look for: 'TBD', 'TODO', 'Similar to Task N', …"）。
这就是 V2。**同样的 token，从会诱导的那一类，搬到能检测的那一类。**

### 反面清单（不要做的事）

- 不要因为"禁令显得严厉、所以更管用"就默认用禁令 —— **微测你自己的场景，别假设**。
- 不要用**猜测**代替分类。反例 #2 在 15 个样本里**根本诱发不出那个失败**，
  于是无从判定 —— 结论是"维持原样、平局选短"，不是"改成配方"。
- 不要只跑程序化打分就下结论：**每一个命中都人工复核**
  （"引用禁令"与"违反禁令"用自动化极易混淆）。
- 不要在没有**无指导对照组**的情况下宣称某措辞有效 —— 反噬只能靠对照组发现。

---

## 八、与本技能包的关联

`writing-skills/SKILL.md` 里那三件套，各自的归类与"为什么这样写是对的"：

| 本包手法 | 类别 | 为什么这样写是对的 |
|---|---|---|
| **Red Flags 表** | **Recognition table** | 它的读取时机是**决策时**（agent 对照"我现在是不是在犯这个形态"），而非组装时。原文 doctrine #2：**Recognition tables work** —— 审计中 **14 个 recognition table 全部 keep**。Red Flags 表以"症状 → 停下"的形态出现，正是"在决策点被查"的正确形态，所以它放在那里不会反噬。 |
| **Common Rationalizations 表** | **Recognition table** | 同上。它列的是**借口**（"Skill is obviously clear" 等），用途是**在 agent 说出该借口的时刻被查到并驳回**——这是 decision-time recognition，不是 composition shaping。doctrine #2 覆盖它，审计中属于那 14 个 keep 之一。原文明确："Red-Flags/rationalization tables read at **decision time**, not composition time." |
| **Iron Law**（全大写禁令） | **Discrete-directive prohibition** + **Policy gate** | "You MUST stop"、全大写"不许跳过测试"这一层，是**治理性/纪律性规则**，不是对"产出该长什么样"的组装塑形；对应 doctrine #3：**Discrete-directive prohibitions work —— 当模型对 "do Y" 没有竞争性激励**。跳过测试、跳过部署清单，**没有**能自圆其说的"帮忙"理由，所以判据成立，因此这类大写禁令**有效**。 |

### 本包已经写对的地方（与原文实测一致）

- `SKILL.md` 的 **Match the Form to the Failure** 表已经把这条规律写进了
  "wrong form" 列：形状类失败对应 `Prohibition list ("don't restate", "never narrate")` 是错的。
- `SKILL.md` 的 **No nuance clauses** 与 **Exemption clauses don't scope** 两条，
  与原文 doctrine #4 的 "adding nuance clauses to a winning recipe makes it worse"、
  以及实测 **3.0 → 3.8** 完全对应。
- Bulletproofing 段开头那句**适用范围声明**
  （"本套工具针对**纪律失败**；对'形状错误'或'漏要素'，禁令式加固会倒退"），
  正是 doctrine #3/#4 的分界线。

### 本包还没写进 SKILL.md 的两条

1. **平局选短（Ties go to the shorter phrasing）** —— 措辞长度是真实成本，
   两种写法实验上等效时留短的。
2. **好 token 在坏位置 → 搬位置，不要删**（V2 手法）—— 从组装期搬到复查期。

---

## 附：数字速查

| 场景 | 措辞 | 数字 | 含义 |
|---|---|---|---|
| Dispatch composition | prohibition | **4.4** | 比无指导（**3.6**）更差 —— 反噬 |
| Dispatch composition | positive recipe | **3.0, zero variance** | 采纳 |
| Dispatch composition | recipe + nuance | **3.8, noisy** | nuance 稀释配方 |
| Test-rerun directive | prohibition | **0/5** 违规（对照组 **3/5**） | 有效 |
| Test-rerun directive | positive recipe | **0/5** | 等效，但更长 → 留短的 |
| 审计规模 | — | ~**30** skills；3 tripwire / 14 recognition table / ~20 policy gate / **5 composition-prohibition** | — |
| 反例 #2 微测 | — | **15 samples** 未能诱发失败 | 无证据，维持原样 |
| writing-plans Stage 1 | — | **0/20** placeholders（含无指导对照组） | 无需改动 |
| writing-plans Stage 1b | — | **40/40** clean（10 任务、强制压力） | 无需改动 |
| 微测成本 | — | **$0.15–0.30/sample** vs 完整评测 **$12/50min** | 先微测，结构性改动才全量确认 |
