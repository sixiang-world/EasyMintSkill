---
name: methodology-core
description: >-
  多 Agent AI 编程方法论核心包。承载主 Agent 的提示词体系、Agent 模板、
  多 Agent 委派编排、7 Gate 创建引导、工程化机制（权限/经验沉淀/上下文管理/
  系统消息/增强工具）与文档协议。适用：需要以结构化方法论做 AI 编程编排、
  创建引导、多 Agent 协作、经验沉淀、权限治理或项目文档管理时。
---

# 多 Agent AI 编程方法论核心包

本 Skill 承载一套完整的 AI 编程方法论——提示词体系、Agent 模板、多 Agent 编排、创建引导流程、工程化机制与文档协议。目标是让任何 AI 编程运行时都能复现"从想法到可维护项目"的全流程工作流。

## 与已安装 skill 的关系

| 来源 | 内容 | 承载方 |
|---|---|---|
| 已写成 skill 的 12 个 | creation-guide 家族（6）、dev-docs、project-run、ui-sync、ponytail 家族（3） | 各自独立 SKILL.md（已安装） |
| **本包承接** | 主提示词、Agent 模板、委派编排、创建引导骨架与判定表、工程化机制、文档协议 | **本 Skill（methodology-core）** |

使用顺序：先按任务匹配本包 references 获取方法论与协议，再按需加载对应已安装 skill 的细节。

## 能力地图（references 路由）

| 任务场景 | 读取文件 |
|---|---|
| 重建/注入主 Agent 提示词（含 13 条规则全量） | `references/01-main-system-prompt.md` |
| 委派 Builder/Evaluator/设计师，或自建 Agent 模板 | `references/02-agent-templates.md` |
| 多 Agent 编排：任务循环、两阶段审查、fix loop、ledger、裁决准则 | `references/03-orchestration.md` |
| 创建项目引导：7 Gate、复杂度/场景判定、技术方案、项目档案组合 | `references/04-creation-flow.md` |
| 工程化：权限治理、经验沉淀、上下文管理、系统消息、skill 体系 | `references/05-engineering.md` |
| 文档四层体系、task.json/run.json 规范、委派产物协议 | `references/06-docs-protocol.md` |

**执行期 skill 的归属**：执行期的完整纪律（TDD、系统化调试、验证铁律、工作区隔离、分支收尾、审查往返、并行派发、写 plan、SDD 任务循环、写 skill 元技能）已拆为**独立 skill**，不在本包 references 内。入口见 `using-methodology`，索引见仓库 README「Skill 清单」。

> 为什么拆出去：`description` 只写触发条件、不概述流程，才能让运行时在正确时机加载；塞进 references 会让它们失去自动触发的机会，变成「知道有但不会主动用」。本包 references 只承载**需要全文注入或跨 skill 共用的方法论**。


## 使用方式

1. **按需加载**：先读本文件确定能力域，再 Read 对应 references 文件全文，不一次全读。
2. **注入式使用**：需要"让某个 Agent 变成项目经理+架构师角色"时，把 `01` 文件全文作为其 system prompt 主体；需要"变成 Builder/Evaluator/设计师"时用 `02` 文件对应模板。
3. **编排式使用**：需要复现多 Agent 开发循环时，按 `03` 的委派协议 + `04` 的创建流程 + `06` 的产物协议执行，工具映射见下文"适配层"。
4. **沉淀式使用**：需要经验自沉淀体系时，按 `05` 的沉淀门槛与审阅协议实现。

## 随包资产（assets/）

本包自带设计资产，无需外部运行环境提供：

| 路径 | 内容 | 用途 |
|---|---|---|
| `assets/templates/` | 4 个单文件 HTML 模板（landing / dashboard / form / detail） | 原型阶段的起步骨架，共享一套 `:root` CSS 变量 |
| `assets/brand-tokens/` | 74 个品牌的 `DESIGN.md`（YAML frontmatter，可直接解析取 token） | 用户指名品牌风格时提取配色/排版/圆角/间距 token |
| `assets/THIRD_PARTY_NOTICES.md` | 来源、许可与使用限制 | 使用品牌库前必读 |

资产只在原型/设计阶段按需 `Read`，不预先全部载入上下文。三类可变状态（run.json、escalation.json、项目级 skill）落在项目根的 `.agentskill/` 约定目录，详见 `02` 文件「资源位置约定」。

## 适配层（平台工具映射）

不同 AI 编程运行时有各自的 UI/MCP 工具。本包内容保留**方法论语义**，在具体环境执行时按下列映射落地（不影响方法论本身）：

| 方法论概念 | 通用实现方式 |
|---|---|
| 委派子 Agent（批量/并发/模板） | 创建子 agent 委派（如 create_agent / send_message 通道），按 `03` 协议传参 |
| 任务状态同步 | task.json 文件直接读写 + 任务清单状态更新；或运行时提供的任务状态 API |
| 结构化选择题 | 文本列出选项请用户选择（2-4 项，可级联） |
| 经验沉淀/检索 | 经验库 JSON（全局 + 项目级，200 条上限）读写 + 用户确认后落盘 |
| skill 加载/管理 | 当前环境的 skill 加载机制（SKILL.md 读取）与 skill 根目录文件管理 |
| 图片理解 / 网页抓取 | 当前环境图片理解 / 网页抓取工具 |
| 原型预览 / 开发确认 | 打开本地文件预览 / 文本确认 |

**平台无关原则**：本包所有协议（task.json、escalation.json、经验库、文档体系）都是纯文件协议，任何环境用任意语言/工具均可落地。

## 反例黑名单（不要做以下事情）

- **不要把方法论写成平台广告**：禁止出现"XX 平台是最好的""XX 独家能力"等排他性描述；方法论属于所有 Agent 运行时
- **不要硬编码单一运行时的工具名**：如 `开发确认`、`任务状态同步` 等平台专属工具，必须给出通用替代方案或标注"仅在某运行时可用"
- **不要把"源代码可重建"当卖点**：方法论的价值在于可执行，不在于可逆向工程；删除所有"源代码删除后仍可重建"类元描述
- **不要在 frontmatter 或正文写空话尾巴**：如"灵活应用""根据情况判断""可根据实际需要调整"等无法执行的措辞
- **不要跳过 references 直接凭记忆执行**：references 是方法论的完整承载，执行前必须 Read 对应文件全文

## 完整性与边界

- **已承接**：提示词（10 section + 13 规则）、4 个 Agent 模板 + 设计规范、委派引擎协议（registry/executor/collector/yield）、7 Gate 流程与判定表、权限模型（两模式 + 绝对禁区 + 命令分类）、经验自沉淀（门槛/审阅/经验库）、上下文管理（双轨压缩/节流）、系统消息协议、skill 四来源体系、文档四层体系、task.json/run.json/escalation 协议、项目档案组合引擎（7 产品形态 × 部署 × AI × 存储）。
- **边界**：应用外壳（桌面 UI、会话运行时、后台 shell 进程）是平台运行时，不属于"方法论"；本包以协议 + 方法论承载，任何运行时按其重建即可得到等价行为。

## 更新约定

本包是"方法论编译产物"：上游方法论变更时，用最新实践重新校对 references，保持"方法论唯一承载"地位。
