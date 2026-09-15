---
name: easymint-core
description: >-
  EasyMint 全量 AI 能力承接包（唯一真相源）。承载该项目未写成 skill 的全部内容——Mint
  系统提示词与 13 条规则、四个 Agent 模板（Mint/Builder/Evaluator/Mint-D）、多 Agent
  委派编排、7 Gate 创建引导、工程化机制（权限/learn 沉淀/上下文管理/系统消息/增强工具）、
  文档四层协议。源代码删除后仍可据此完整重建等价能力。适用：需要以 EasyMint 方式做
  AI 编程编排、创建引导、多 Agent 协作、经验沉淀、权限治理或项目文档管理时。
---

# EasyMint Core — 全量能力承接包

本 Skill 是开源项目 **EasyMint**（内置 Pi Coding Agent 的桌面 AI 编程平台）全部 AI 能力的**唯一真相源**。
它承接了该项目里**没有写成 skill 的内容**——提示词体系、Agent 模板、多 Agent 编排、创建引导流程、工程化机制与文档协议。

**承诺**：删除 EasyMint 源代码后，仅凭本 Skill 即可完整重建其 AI 行为与工作流能力。

## 与已安装 skill 的关系

| 来源 | 内容 | 承载方 |
|---|---|---|
| 已写成 skill 的 12 个 | creation-guide 家族（6）、dev-docs、project-run、ui-sync、ponytail 家族（3） | 各自独立 SKILL.md（已安装） |
| **未写成 skill 的（本包承接）** | Mint 系统提示词、13 条规则、Agent 模板、委派编排、创建引导骨架与判定表、工程化机制、文档协议 | **本 Skill（easymint-core）** |

使用顺序：先按任务匹配本包 references 获取方法论与协议，再按需加载对应已安装 skill 的细节。

## 能力地图（references 路由）

| 任务场景 | 读取文件 |
|---|---|
| 重建/注入 Mint 主提示词（含 13 条规则全量） | `references/01-mint-system-prompt.md` |
| 委派 Builder/Evaluator/Mint-D，或自建 Agent 模板 | `references/02-agent-templates.md` |
| 多 Agent 编排：何时委派、task 协议、结果收集、并发与上下文保护 | `references/03-orchestration.md` |
| 创建项目引导：7 Gate、复杂度/场景判定、技术方案、项目档案组合 | `references/04-creation-flow.md` |
| 工程化：权限治理、learn 经验沉淀、上下文管理、系统消息、skill 体系 | `references/05-engineering.md` |
| 文档四层体系、task.json/run.json 规范、委派产物协议 | `references/06-docs-protocol.md` |

## 使用方式

1. **按需加载**：先读本文件确定能力域，再 Read 对应 references 文件全文，不一次全读。
2. **注入式使用**：需要"让某个 Agent 变成 Mint"时，把 `01` 文件全文作为其 system prompt 主体；需要"变成 Builder/Evaluator/Mint-D"时用 `02` 文件对应模板。
3. **编排式使用**：需要复现多 Agent 开发循环时，按 `03` 的委派协议 + `04` 的创建流程 + `06` 的产物协议执行，工具映射见下文"适配层"。
4. **沉淀式使用**：需要经验自沉淀体系时，按 `05` 的 learn-gate 门槛与 learn 协议实现。

## 适配层（平台工具映射）

EasyMint 原平台有专属 UI/MCP 工具（show_confirm_dev、set_task_status、show_prototype、ask_user 等）。本包内容保留**原文语义**，在其他环境执行时按下列映射落地（不影响方法论）：

| EasyMint 原工具 | 本环境等价实现 |
|---|---|
| `task`（委派子 Agent，批量/并发/模板） | 创建子 agent 委派（如 create_agent / send_message 通道），按 `03` 协议传参 |
| `set_task_status` / `refresh_tasks` | task.json 文件直接读写 + 任务清单状态更新 |
| `ask_user`（结构化选择题） | 文本列出选项请用户选择（2-4 项，可级联） |
| `learn` / `search_experiences` | 经验库 JSON（全局 + 项目级，200 条上限）读写 + 用户确认后落盘 |
| `use_skill` / `manage_skill` | 本环境的 skill 加载机制（SKILL.md 读取）与 skill 根目录文件管理 |
| `describe_image` / `web_fetch` | 当前环境图片理解 / 网页抓取工具 |
| `show_prototype` / `show_confirm_dev` | 打开本地文件预览 / 文本确认 |

**平台无关原则**：本包所有协议（task.json、escalation.json、经验库、文档体系）都是纯文件协议，任何环境用任意语言/工具均可落地。

## 完整性承诺与边界

- **已承接**：提示词（10 section + 13 规则）、4 个 Agent 模板 + 设计规范、委派引擎协议（registry/executor/collector/yield）、7 Gate 流程与判定表、权限模型（两模式 + 绝对禁区 + 命令分类）、learn 自沉淀（门槛/审阅/经验库）、上下文管理（双轨压缩/节流）、系统消息协议、skill 四来源体系、文档四层体系、task.json/run.json/escalation 协议、项目档案组合引擎（7 产品形态 × 部署 × AI × 存储）。
- **边界**：应用外壳（Electron UI、Pi SDK 会话运行时、后台 shell 进程）是平台运行时，不属于"AI 能力"；本包以协议 + 方法论承载，删源码后按其重建即可得到等价行为。

## 更新约定

本包是"编译产物"：EasyMint 上游变更时，用最新源码重新校对 references（特别关注 `app/shared/prompts.ts` 与 `app/main/services/` 下的协议实现），保持"唯一真相源"地位。
