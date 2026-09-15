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

- 委派失败：重试 ≤ 3 次 → Builder 写 `.agent-config/escalation.json` → 主 Agent 汇报原因和选项（重试/跳过/人工介入）。
- escalation.json 协议：`{ type: "escalation", from, taskId, reason, details, options: ["重试","跳过","人工介入"], timestamp }`。
- decision.json 协议：主 Agent 在用户决策后写入 `{ taskId, action: "retry"|"skip"|"abort", reason?, timestamp }`，然后继续任务执行。
- **进度监控者原则**：主 Agent 每轮自行核实真实进度（读 task.json / git diff / escalation.json / 代码），不盲信 status 字段——凭代码现状判断该重做/验收/跳过。

## 七、委派循环标准流程（主 Agent 视角）

```
1. 读 task.json + 开发记录，核实真实进度
2. 按依赖顺序找下一个未完成任务（以核实状态为准）
3. 任务状态同步(id, "building") → Task(agent="builder", taskId=id)  ← 不转述任务全文
4. Builder 完成 → 任务状态同步(id, "evaluating") → Task(agent="evaluator", taskId=id)
5. 验收通过 → 状态自动回写 done → 更新开发记录快照与当日明细 → 下一任务
6. 失败 → 重试 ≤3 → escalation.json → 汇报选项
7. 全部完成 → 生成/更新 .agent-config/run.json → 简要总结
```
