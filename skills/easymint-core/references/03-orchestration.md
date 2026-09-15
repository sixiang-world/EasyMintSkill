# 多 Agent 编排（03）

> 本文件承载多 Agent 编排的完整协议。
> 本文件承载"三权分立 + 委派引擎"的完整协议。任何环境复现多 Agent 开发循环时，按此实现。

## 一、角色分工

| 角色 | 职责 | 会话关系 |
|---|---|---|
| **主 Agent**（主会话） | 项目经理 + 架构师：需求理解、拆解、调度、进度监控、向用户汇报 | 唯一与用户对话的角色 |
| **Builder** | 按 task.json 任务写代码（tdd 先红后绿、lint+build、git commit、3 次失败写 escalation.json） | 独立会话，看不到主对话历史 |
| **Evaluator** | 验收 Builder 产出（按项目类型：Web 用 Playwright 截图+交互验证；非 Web 用测试+curl+代码审查） | 独立会话，输出 PASS/FAIL |
| **设计师 Agent** | UI 设计，产出 HTML 原型 | 独立会话，产出即止；预览/反馈归主 Agent |
| **标准白板子 Agent** | 查资料、读代码、分析、跑验证——通用委派，无模板人设 | 独立会话，回传结果/摘要 |

## 二、委派决策树（上下文保护是核心）

委派的目的不是分工，而是**防止大量消耗 token / 占用上下文的任务撑满主会话，让你失去对项目的整体掌控**。分场景判断，不默认委派也不默认亲自。

```
收到任务
├─ 必须委派（上下文保护）：
│   ├─ 要读/探索大量内容后产出结论（跨文件、深调研）→ 委派白板子 agent，回传摘要
│   ├─ 会产生海量中间输出（长测试、大日志、批处理、全量扫描）
│   └─ 判据：「这些中间内容进主会话还用不用得到？」用不到/只要结论 → 委派
├─ 必须亲自（综合与掌控）：
│   ├─ 方案最终综合、决策、需求理解（Never delegate understanding）
│   └─ 项目走向判断、对用户反馈的直接响应（对话本身）
├─ 按上下文重合度：
│   ├─ 强依赖当前会话上下文 → 亲自（拆出去增加同步成本）
│   ├─ 独立可并行需要多样性（多方案对比/对抗审查/并行探索）→ 委派
│   └─ 简单任务（改几行/查报错/单文件读）→ 亲自
└─ 硬约束：
    ├─ 委派深度 = 1：子 agent 不得再委派
    └─ 委派类型化：明确「探索型/审查型/实现型」
```

## 三、task 工具协议

### 参数

| 参数 | 说明 |
|---|---|
| `agent` | 可选模板名（builder/evaluator/designer/自定义）；省略 = 标准白板子 Agent |
| `model` / `provider` | 可选覆盖子 Agent 模型（委派指定 > 模板 > 子 Agent 默认 > 全局） |
| `description` | 任务简述（单任务模式） |
| `prompt` | 详细任务指令（单任务模式） |
| `taskId` | 关联的 task.json 任务 id——委派完成/中止时**自动回写** done/failed，任务配置面板实时同步 |
| `outputSchema` | 结构化输出格式；子 Agent 必须调 yield 工具按此格式返回 |
| `tasks` | 批量任务数组（每个可指定不同 agent/model/prompt/taskId/outputSchema） |
| `readOnly` | 只读模式（验收/审查场景）——子 Agent 只配读工具 |
| `concurrency` | 批量并发数，默认 4 |

### 关键行为

1. **异步委派**：tool.execute 创建委派记录后**立即返回**「已启动 N 个子 Agent 执行，完成后结果将注入会话」——不阻塞主 Agent 模型循环。
2. **结果注入**：委派完成经 onComplete 回调以系统消息注入主会话（触发新回合让主 Agent 自动总结）。
3. **单任务即时通知**：批量中单个任务完成/被用户停止 → 立即注入通知（不等整个委派收尾）；文本明确「已由用户中断」防主 Agent 误判意外失败自动重启。
4. **task.json 逐任务即时回写**：任务一进入终态立即 done/failed，不等委派整体收尾。
5. **状态回写规则**：building/evaluating 由主 Agent 手动调 任务状态同步；done/failed 由委派结果自动回写，手动标记终态会被拒绝。

### 结果注入格式（formatDelegationResult）

```
摘要段（前端渲染绿色结果气泡）：
[批量] 共 N 个子任务: X 成功, Y 失败
⏺ <标题> — 完成 · 12s
⏺ <标题> — 失败

详细段（主 Agent 汇报用）：
详细结果:
<标题>:
<output 前 2000 字符>
---
结构化结果: {JSON}
```

## 四、执行引擎（registry + executor）

### 委派记录（registry）

- `createDelegation`：创建记录（含 AbortController、每任务独立中止控制器、completion Promise），立即返回。
- **临时/真实会话 ID 双匹配**：新会话绑定临时 UUID，createPiSession 返回真实 ID 后注册映射；steer/abort 按两者都能定位。
- `abortDelegations(parentSessionId)`：用户打断 → 中止该主会话全部运行中委派。
- `abortTask(delegationId, index)`：进度条单任务停止。
- 保留最近 50 条已完成记录供查询/调试。

### 子 Agent 执行（executor）

1. **目录分级**：`<项目会话目录>/<主会话ID>/subagents/`——子会话归属清晰，不与主会话平级。
2. **并发**：`mapWithConcurrencyLimit`，默认并发 4；整体 abort signal 传播。
3. **工具集**：readOnly → 只读工具集；否则基础工具 + 增强 edit（diff 注入返回文本）。子 Agent 权限跟随主会话（同权限模式 + 绝对禁区）。
4. **输出截断**：超过 MAX_OUTPUT_LINES（5000 行）或 MAX_OUTPUT_BYTES（500KB）→ 截断并标注 `[输出已截断]`（截尾保留）。
5. **yield 结构化输出**：有 outputSchema 时注入 yield 工具（data 必须符合 schema），完成工作后调用；执行结束统一组装为 structuredOutput。
6. **进度事件**：tool_execution_start（currentTool/toolCount）、message_end（tokens/requests）、auto_retry_start/end（retryState/retryFailure）、model 切换追踪；200ms 节流回调 onProgress。
7. **失败兜底**：单任务抛异常/会话创建失败 → 必须 finishDelegation（否则 completion 永不 resolve、卡片永久 running）；单任务失败不阻塞其他任务。
8. **思考等级**：模板设置 > 父会话等级 > medium，再按子 Agent 模型能力自适应（同主会话「同等级→向下→向上」规则）。
9. **模型解析优先级**：委派指定(provider+model) > AgentTemplate > 子 Agent 默认模型(settings) > 全局默认。

### 结果收集（collector）

按消息 id 替换（杜绝累积快照拼接重复）——同一消息的多次快照只保留最新，最终 getText() 输出完整对话文本。

## 五、系统消息协议（委派注入）

- 注入方式：`injectSystemMessage(sid, text, kind, { triggerTurn })`。
- kind：delegation / shell / 项目初始化信号 / 直接创建信号 / flow / handoff / summary / learn。
- **triggerTurn 规则**：委派整体完成 → true（开回合让主 Agent 总结）；事件通知 → false（不经回合）；单任务被停止且无后续 → true。
- 内容前缀 `[系统消息]` 是模型识别的唯一依据（SDK 把 custom 映射为 user 角色，模型看不到 customType）。

## 六、中断恢复与 escalation

- 委派失败：重试 ≤ 3 次 → Builder 写 `.agentskill/escalation.json` → 主 Agent 汇报原因和选项（重试/跳过/人工介入）。
- escalation.json 协议：`{ type: "escalation", from, taskId, reason, details, options: ["重试","跳过","人工介入"], timestamp }`。
- decision.json 协议：主 Agent 在用户决策后写入 `{ taskId, action: "retry"|"skip"|"abort", reason?, timestamp }`，然后继续任务执行。
- **进度监控者原则**：主 Agent 每轮自行核实真实进度（读 task.json / git diff / escalation.json / 代码），不盲信 status 字段——凭代码现状判断该重做/验收/跳过。

## 六之二、任务循环的完整闭环（superpowers 吸收）

> 本节把上节的「重试 ≤ 3 → 问人」升级为完整的自驱闭环。核心目标：**绝大多数问题由主 Agent 自己裁决推进，只在四种不可逆/越界情况下停下来问人。**

### 6.2.1 子 Agent 的四种报告状态

委派结果必须收敛到这四种状态之一：

| 状态 | 含义 | 主 Agent 动作 |
|---|---|---|
| `DONE` | 完成，无疑虑 | 进入审查 |
| `DONE_WITH_CONCERNS` | 完成，但有保留意见 | 进入审查，**保留意见作为审查输入** |
| `NEEDS_CONTEXT` | 信息不足，无法开始 | 补上下文后**同模型**重派 |
| `BLOCKED` | 受阻，无法继续 | 四路分流（见下） |

**BLOCKED 的四路分流**（不得原样重试——子 Agent 说卡住，就说明有东西必须变）：

1. 是上下文问题 → 补足上下文，同模型重派
2. 需要更强推理 → 换更强模型重派
3. 任务太大 → 拆成更小的任务
4. 方案本身错了 → 裁决修正方案，记账，携带裁决重派

> **Never** ignore an escalation or force the same model to retry without changes. If the implementer said it's stuck, something needs to change.
> （绝不忽视升级信号，也绝不让同一模型不做任何改变地重试。）

### 6.2.2 返工循环（fix loop）：5 轮上限

**触发条件**：审查返回规格 ❌、任一 Critical/Important 问题，或主 Agent 确认为真实缺口的 ⚠️ 项。

**不触发的情况**（重要）：
- **Minor 问题一律不进循环**，记账为 `Task <N>: minor (deferred): <一句话>`，交给终局全分支审查统一分诊。
  > 注意：集中记录却无人查看 = 静默丢弃。终局审查必须真的处理这些条目。
- **方案强制的冲突**：不得因为「方案就是这么要求的」而直接驳回问题，也不得在无裁决记录的情况下派发一个与方案矛盾的修复。由主 Agent 裁决后才行动。

**循环规格**：
- 一轮修复 = 一次修复派发 + 一次**范围化复审**
- **每个任务最多 5 轮**
- **第 1-3 轮**：续用原实现者（它的上下文完好，知道任务、代码和自己的选择）
- **第 4-5 轮**：派新实现者 + 至少高一档能力模型
  > A loop that survives three resumes usually means the implementer cannot see its own problem — fresh eyes and a capability bump in one move.
  > （能扛过三次续用的循环，通常意味着实现者看不到自己的问题——新视角和能力提升一次到位。）

**每轮强制证据**：修复者修完后要重跑**覆盖被改代码的测试**，把修复报告追加到同一份报告文件，并回传 short contract。派复审前必须确认修复报告含【覆盖测试 + 运行的命令 + 输出】三项；三项齐了才派复审。（一行修复不需要跑全量套件，但要指名覆盖它的测试文件。）

### 6.2.3 Breaker：第 5 轮后的三路裁决

到达上限后不再继续修，而是对每个遗留问题做裁决：

| 情形 | 处理 |
|---|---|
| **审查者错了，或该点可争议** | 搁置：`Task <N>: parked — <问题> — Ruling: <为什么代码就该这样>`。终局审查会看到双方论点。 |
| **确实真，但下游没有东西依赖它** | 同样搁置，但裁决要写明「这是真的，且已延后」。 |
| **确实真，且是承重的**（后续任务建立在它之上，或它暴露了方案缺陷） | 裁决**最小改动**以解锁依赖方，记账为 `Task <N>: Ruling: <问题> — <你决定了什么、为什么>`，并**带进下一个任务的派发**。静默搁置结构性失败，会让所有依赖它的任务都建在坏地基上。 |
| **只有当前进路线全是猜测时才停下问人** | 例外情形，停下来。 |

> Adjudicate only at the cap. Adjudicating earlier to end a loop is pre-judging with a different name. Every adjudication is a ledger entry — **a silent discard is forbidden.**
> （只在到达上限时裁决。提前裁决以求结束循环，只是换了个名字的预判。每次裁决都必须记账——**禁止静默丢弃。**）

### 6.2.4 Ledger（记账本）：压缩后唯一的恢复地图

`Ledger` 是一个 append-only 的进度账本，落在 `.agentskill/sdd/<plan-basename>/ledger.md`。**它是上下文被压缩后唯一能还原进度的东西**——没有账本的主 Agent 会重复派发已完成的整个任务序列。

固定行格式（原样照抄这几种）：

```
Task <N>: fix round <R>/5 (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)
Task <N>: complete (commits <base7>..<head7>, review clean)
Task <N>: complete (commits <base7>..<head7>, <K> parked)
Task <N>: minor (deferred): <one-liner>
Task <N>: parked — <finding> — Ruling: <why the code stands>
Task <N>: Ruling: <finding> — <what you decided and why>
```

### 6.2.5 Rulings, not stalls（做裁决，不停摆）

> **Rulings, not stalls.** A running plan does not wait on a human. Conflicts, ambiguities, plan defects, a cap you would have asked to exceed — decide them. The spec is the binding authority, the plan is its argument, and your judgment settles what neither answers. Record every decision in the ledger as `Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep going. A wrong ruling costs rework your human partner can see and undo; a session parked on a question costs their whole day and buys nothing.
>
> （**做裁决，不停摆。** 一个正在推进的计划不等人类。冲突、歧义、方案缺陷、你想申请突破的上限——都自己裁决。需求文档是约束性权威，方案是它的论证，你的判断裁定两者都没回答的部分。每个决定都记进账本：`Ruling: <决定了什么> — <为什么> — <如果错了的代价>`，然后继续推进。一个错误裁决的代价是你的用户能看到、能撤销的返工；一个卡在提问上等着的会话，代价是他们一整天，且什么都换不来。）

**四条唯一的停摆条件**（只有这四种才停下问人）：

1. **不可逆或破坏性操作**
2. **安全敏感操作**
3. **工作区之外的副作用**（按惯例需要先问的：合并、推送到共享分支、发布）
4. **方案烂到每条前进路线都只是猜测**

**Continuous execution**：不在任务之间暂停问人，执行完方案里的全部任务。唯一的停止理由就是上面四条，或者全部任务完成。

**禁止"要不要继续"式提问**——那是在浪费用户时间。

**终局 Rulings 穷举回报**：收尾时，把账本里所有含 `Ruling:` 的行按时间顺序收集到最终报告里，标题「我做的裁决」，每条附上「如果错了的代价」。这份清单必须穷举——账本里有就必须出现在清单里。
> A ruling that dies with the workspace was a decision made in secret.
> （随工作区一起消失的裁决，等于一个秘密做出的决定。）

### 6.2.6 派发纪律（上下文卫生）

**派发 prompt 必须包含的五项**：

1. 一句话说明这个任务在项目中的位置
2. **任务简报的路径**，并说明「先读这个——它就是你的需求，里面的精确值要原样使用」
3. 简报无法知道的、来自前序任务的接口与决定
4. 你对简报中发现的歧义所做的裁决
5. 报告文件路径与报告契约

**硬约束**：
- 精确值（数字、魔法字符串、签名、测试用例）**只出现在简报里**
- **`Never make a subagent read the whole plan file.`**（绝不让子 Agent 读整个方案文件——用简报把单任务抽出来）
- **`Hand artifacts over as files.`**（产出物用文件交付）

**反粘贴历史**（真实事故教训）：
> A dispatch prompt describes one task, not the session's history. Do not paste accumulated prior-task summaries ("state after Tasks 1-3") into later dispatches — a real session's dispatch hit **42k chars of which 99% was pasted history**. A fresh subagent needs its task, the interfaces it touches, and the global constraints. **Nothing else.**
>
> （派发 prompt 描述的是**一个任务**，不是会话历史。不要把累积的前序任务摘要粘进后续派发——真实事故里，一次派发的 42k 字符中 99% 是粘贴的历史。新子 Agent 只需要：它的任务、它涉及的接口、全局约束。**别的都不要。**）

**为什么**：你粘进派发 prompt 的一切，以及子 Agent 打印回来的一切，都会留在你的上下文里、在后续每个回合被重读。**所以产出物一律走文件。**

**short contract 回传**（子 Agent 只回传这些，15 行以内，细节在报告文件里）：

```
- Status: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
- Commits created（短 SHA + 标题）
- 一行测试摘要（如 "14/14 tests passing, output pristine"）
- 你的保留意见（如有）
- 报告文件路径
```

### 6.2.7 主 Agent 的职责边界

| 主 Agent **做** | 主 Agent **不做** |
|---|---|
| 建工作区、读方案、跑预检冲突扫描、写账本 | **不亲自修问题**——控制器会话里的修复会污染你的上下文，且绕过了审查。要让实现者继续做。 |
| 选模型/能力档位、写派发 prompt、回答实现者提问 | 不并行派发多个实现者（会冲突） |
| 生成审查包、派审查者、裁决、升级、记账 | 不向审查者预判问题（见下） |
| 维护账本、终局汇总所有裁决 | 不在任务间暂停问人 |

**禁止向审查者预判问题**：
> Do not pre-judge findings for the reviewer — never instruct a reviewer to ignore or not flag a specific issue. If you believe a finding would be a false positive, let the reviewer raise it and adjudicate it in the review loop. If the prompt you are writing contains **"do not flag," "don't treat X as a defect," "at most Minor," or "the plan chose"** — stop: you are pre-judging, usually to spare yourself a review loop.
>
> （不要替审查者预判问题——绝不指示审查者忽略或不标记某个问题。如果你认为某个问题会是误报，让审查者提出来，然后在审查循环里裁决它。如果你正在写的 prompt 里出现 **"do not flag" / "don't treat X as a defect" / "at most Minor" / "the plan chose"**——停下：你在预判，通常是为了给自己省一轮审查。）

**禁止子 Agent 递归派发**（双向约束，两个模板里都要有）：
- 实现者不得派子 Agent 做本任务的一部分
- **尤其不得派审查者来检查自己的工作**——自我审查就是读自己的 diff；审查是控制器的工作。自己派出的审查者是重复占位，其通过在本流程里**不计分**。
- 若实现者起念「找个独立审查能强化我的报告」——**那个审查已经安排好了。回去报告。**

## 七、委派循环标准流程（主 Agent 视角）

```
1. 读 task.json + 开发记录 + 账本，核实真实进度
2. 按依赖顺序找下一个未完成任务（以核实状态为准）
3. 生成任务简报文件（从方案抽取单任务全文，不转述）
4. 派发 Builder：给五项要素 + 简报路径 + 报告路径与契约
5. Builder 回传 short contract
   - NEEDS_CONTEXT → 补上下文重派
   - BLOCKED → 四路分流
   - DONE / DONE_WITH_CONCERNS → 继续
6. 生成审查包 → 派 Evaluator（两阶段：规格 + 质量）
7. 审查结果分流：
   - 两个裁决都通过 → 记账 complete → 更新开发记录 → 下一任务
   - 有问题 → fix loop（≤5 轮；1-3 轮续用，4-5 轮换新+更强模型）
   - 到上限 → breaker 三路裁决 → 记账 → 继续
   - Minor → 记账 deferred → 不进循环
8. 全部任务完成 → 终局全分支审查（最强模型 + requesting-code-review）
   - 有问题：只派**一次**修复（带全部问题）+ 一次范围化复审；**没有第二轮修复波次**
9. 生成/更新 .agentskill/run.json → 汇报（含「我做的裁决」穷举清单）→ 转 finishing-a-development-branch
```

### 关于并发

多任务可并发的前提是**任务间无依赖、无共享状态**。判定见 `dispatching-parallel-agents` skill：
- 同一响应里发多个派发调用 = 并行；分响应发 = 串行
- 派发时明确约束「不要改其他任务的代码」
- 返回后必须检查是否改了同一处代码、跑全量测试、抽查（子 Agent 会犯系统性错误）

**默认串行**。并行是优化手段，不是目标——冲突的代价高于并发的收益。

