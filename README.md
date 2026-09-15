# EasyMint Skill

> 从想法到可维护项目的全流程 AI 开发方法论。4 角色编排 + 7 Gate 创建引导 + 工程化规范，让 AI 开发不只是写代码，而是交付可维护的产品。

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)](https://skills.sh)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Skills](https://img.shields.io/badge/13%20skills-pack-green)]()

## 这是什么？

EasyMint Skill 是一套 **AI 编程 Agent 的全流程开发方法论**。它把"从模糊想法到可维护项目"的完整流程拆成 13 个可独立加载的 Skill，覆盖：

- **多 Agent 编排**：主 Agent（调度）/ Builder（编码）/ Evaluator（验收）/ 设计师 Agent（设计）四角色模板
- **7 Gate 创建引导**：意图采集 → 功能共创 → 成本校验 → 快速原型 → 技术方案 → 开发 → 验证
- **工程化规范**：文档四层协议、task.json 任务管理、run.json 运行配置、任务状态同步
- **质量守护**：Ponytail 反过度工程三兄弟（主规则 / Code Review / 全仓审计）
- **随包设计资产**：74 个品牌设计规范库（可直接解析取 token）+ 4 个单文件 HTML 起步模板

## Skill 清单（27 个）

### 入口层（两层）

| Skill | 用途 |
|---|---|
| `using-easymint` | **调度层**：任何回应或动作前先查 skill；Red Flags 反合理化表；skill 优先级；子 Agent 隔离阀 |
| `kickoff` | **创作起手层**：任何创作性工作前强制触发；三路径分类（Spike/Bounded/Architectural）+ HARD-GATE 批准闸门 + 隐藏复杂度单向升级 + 规格自审 + Visual Companion 可视化伴侣 |

**为什么要两层**：`using-easymint` 解决"该用哪个 skill"，`kickoff` 解决"从哪开始"。只调度不起手，Agent 仍会直接冲进实现 —— 而"简单任务"恰恰是未经检验的假设造成返工最多的地方。

### 创建期（EasyMintSkill 原生优势）

| 分类 | Skill | 用途 |
|---|---|---|
| **核心** | `easymint-core` | 方法论核心包：主 Agent 系统提示词 + 13 条规则 + 4 Agent 模板 + 多 Agent 编排 + 7 Gate 流程 + 工程化机制 + 文档协议 + 品牌库/种子模板 |
| **创建引导** | `creation-guide` | 创建项目总编排：复杂度判定 + 场景识别 + 阶段路由 + 7 Gate 复核 |
| | `creation-flow-intent` | 意图采集：从模糊想法到具体目标，前沿轮次制 |
| | `creation-flow-features` | 功能共创：细化功能范围，P1/P2/P3 排序，切 MVP |
| | `creation-flow-cost` | 成本校验：部署/AI/预算与功能冲突检测，取舍选项 |
| | `creation-flow-prototype` | 快速原型：HTML 原型产出，渲染正确性审查，分档修改 |
| | `creation-flow-techspec` | 技术方案：三重验证 + 环境就绪 + 落盘任务清单 |

### 计划期与执行期（吸收 superpowers）

| 分类 | Skill | 用途 |
|---|---|---|
| **计划** | `writing-plans` | 任务拆解：Task 层最小可验收单元 / Step 层 2-5 分钟单动作；每任务必备 Files·Interfaces·Run+Expected；No Placeholders 禁令 |
| | `executing-plans` | 同会话批量执行 + 遇阻塞才停（无 subagent 时的降级路径） |
| **执行编排** | `subagent-driven-development` | **王牌机制**：每任务新 subagent + 两阶段审查（规格/质量）+ fix loop 5 轮上限 + ledger 记账 + breaker 三路裁决；含 3 个 prompt 模板与 3 个脚本 |
| **实现纪律** | `test-driven-development` | RED-GREEN-REFACTOR 铁律；无失败测试不写生产代码；含测试反模式参考 |
| | `systematic-debugging` | 四阶段根因定位（根因/模式/假设/实现）；3+ 次修复失败 = 架构问题；含根因追溯·纵深防御·条件等待三份技术参考 |
| **收口** | `verification-before-completion` | Iron Law：无新鲜验证证据不得声称完成；Gate Function 五步；禁止"应该没问题" |
| | `requesting-code-review` | 何时请求审查、四分类处置反馈、终局全分支审查模板（`references/code-reviewer.md`） |
| | `receiving-code-review` | **"不盲目服从"机制**：六步响应、零感谢语、六种可驳回情形、YAGNI 反制 |
| **工作区** | `using-git-worktrees` | 隔离工作区：先检测已有隔离、原生工具优先、git 回退；测试基线校验 |
| | `finishing-a-development-branch` | 收尾：验证测试 → 检测环境 → 三选项菜单（合并/PR/保留）；discard 需精确确认 |
| **并发** | `dispatching-parallel-agents` | 独立问题域并发派发：并发判定决策树、避免冲突、同响应多派发即并行 |

### 工程化与元能力

| 分类 | Skill | 用途 |
|---|---|---|
| **工程化** | `dev-docs` | 项目文档规范：导航页 / 日期明细 / CHANGELOG / 技术架构四层 |
| | `project-run` | 运行配置：`run.json` 格式规范 + 脚本管理 |
| | `ui-sync` | 开发中的任务状态同步：任务清单状态与真实进度同步 |
| **质量守护** | `ponytail` | 反过度工程主规则：最懒可行方案，YAGNI，stdlib 优先 |
| | `ponytail-review` | 过度工程 Code Review：diff 级别的复杂度审计 |
| | `ponytail-audit` | 全仓过度工程审计：ranked list of what to delete/simplify |
| **元技能** | `writing-skills` | **写 skill 即 TDD**：RED-GREEN-REFACTOR、反合理化四件套、Match the Form to the Failure、三份重型参考下沉 |

## 完整工作流

从想法到可维护项目的全链路：

```
using-easymint（调度层：先查 skill 再动手）
  │
  └─ kickoff（创作起手层：分类 → 澄清 → 设计 → 拿批准）
       │
       ├─ Spike ──────────→ 探针调查 → 汇报建议（终点，不写文档）
       │
       ├─ Bounded ────────→ 对话内简短设计 → 用户批准 → 常规开发流程
       │
       └─ Architectural ──→ 澄清 → 2-3 方案 → 分节设计 → 写规格 → 规格自审 → 用户评审
            │
            ├─ 创建期（EasyMintSkill 原生：把模糊想法变成明确方案）
            │   creation-guide ──┬→ creation-flow-intent     意图采集
            │                    ├→ creation-flow-features   功能共创 + 切 MVP
            │                    ├→ creation-flow-cost       成本校验
            │                    ├→ creation-flow-prototype  快速原型 + 渲染审查
            │                    └→ creation-flow-techspec   技术方案 + 环境就绪
            │
            ├─ 计划期（吸收 superpowers：把方案变成可执行任务）
            │   writing-plans ──→ 任务拆解（Task 层 + Step 层，含 Files/Interfaces/验证）
            │
            └─ 执行期（吸收 superpowers：带纪律地推进）
                using-git-worktrees（隔离工作区 + 测试基线）
                     ↓
                subagent-driven-development（每任务新 subagent + 两阶段审查）
                     ├→ test-driven-development       实现纪律
                     ├→ systematic-debugging          出 bug 时
                     ├→ dispatching-parallel-agents   独立问题域并发
                     └→ requesting/receiving-code-review  审查往返
                          ↓
                     verification-before-completion   声称完成前
                          ↓
                     finishing-a-development-branch   收尾（合并/PR/保留）

  贯穿：ponytail 三兄弟（反过度工程）· dev-docs（文档）· project-run（运行配置）
         kickoff 的 Visual Companion（对话中展示 mockup / 架构图 / 方案对比）
```

**贯穿全流程的质量机制**（每个 skill 都配）：

- **Red Flags 表**：预判"这个任务很简单不用走流程"这类偷懒念头，出现即停手查 skill
- **Common Rationalizations 表**：把借口逐条驳回（如"测试后补也一样" → "测试后写的永远通过，证明不了任何事"）
- **Iron Law**：TDD / 调试 / 验证各有全大写单行铁律，用代码块包裹以示不可协商

## 执行期核心机制速览

`subagent-driven-development` 是整个执行期的引擎，其关键设计：

| 机制 | 作用 |
|---|---|
| **两阶段审查** | 一个审查者读一次 diff、返回两个裁决（规格符合性 + 代码质量）。不是两个 subagent——省成本而不损信息 |
| **fix loop 5 轮上限** | 1-3 轮续用原实现者（上下文完好）；4-5 轮换新实现者 + 更高能力档位（"新视角和能力提升一次到位"） |
| **breaker 三路裁决** | 到上限后不再修：审查者错了→搁置；真但不承重→搁置写清；真且承重→裁决最小改动并带进下游任务 |
| **ledger 记账本** | append-only，是上下文压缩后**唯一**的恢复地图。没有账本会重复派发已完成的任务序列 |
| **Rulings, not stalls** | 冲突/歧义/方案缺陷自行裁决并记账，不停下来问人。只有四种情况停：不可逆操作、安全敏感、工作区外副作用、方案烂到每条路都是猜测 |
| **文件即接口** | 任务简报/实现报告/审查包/账本全部走文件，避免"42k 字符派发里 99% 是粘贴的历史"这类上下文事故 |
| **禁止预判问题** | 不得指示审查者忽略某问题。出现 "do not flag" / "at most Minor" 这类措辞即是在给自己省一轮审查 |

> 完整机制见 `skills/subagent-driven-development/SKILL.md` 与 `skills/easymint-core/references/03-orchestration.md`。


## 快速开始

### 快速安装

```bash
# 方式：克隆后按所用 runtime 的 skills 目录放置
git clone https://github.com/sixiang-world/EasyMintSkill.git
cp -r EasyMintSkill/skills/* <你的 runtime 的 skills 目录>/
```

> 若该 runtime 提供了 skill 市场或 CLI 安装器（如 `npx skills add`），也可用其安装：把仓库来源指向 `sixiang-world/EasyMintSkill` 即可。是否支持取决于你的工具。

### 安装到其他 Agent Runtime

EasyMint Skill 兼容所有支持 Agent Skills 标准的 runtime。将 `skills/` 目录下的任意 Skill 复制到对应 runtime 的 skills 目录即可：

```bash
git clone https://github.com/sixiang-world/EasyMintSkill.git
cp -r EasyMintSkill/skills/* <你的 runtime 的 skills 目录>/
```

各 runtime 的 skills 根目录不同（例如部分工具用 `~/.claude/skills/`，部分用 `~/.codex/skills/`），请按所用工具的实际约定放置。注意 `easymint-core` 自带 `assets/` 子目录，复制时需**整目录递归复制**，否则品牌库与模板会丢失。

## 随包设计资产

`easymint-core` 自带设计资产，开箱即用、无外部依赖：

| 资产 | 路径 | 说明 |
|---|---|---|
| 种子 HTML 模板 | `skills/easymint-core/assets/templates/` | 4 个单文件模板：`landing.html` / `dashboard.html` / `form.html` / `detail.html`。共享一套 `:root` CSS 变量，无框架无依赖 |
| 品牌设计规范库 | `skills/easymint-core/assets/brand-tokens/` | 74 个知名品牌的设计系统分析，YAML frontmatter 格式，可直接解析提取配色/排版/圆角/间距 token |
| 来源与授权 | `skills/easymint-core/assets/THIRD_PARTY_NOTICES.md` | **使用品牌库前请先阅读**，含商标与字体授权限制说明 |

> 品牌库仅作**风格参考**：请只提取设计语言（色值、字号梯度、间距节奏）作为起点，不要复制品牌名称、商标、logo 或专有字体。

### 触发方式

| 你想说的话 | 触发的 Skill |
|---|---|
| **"我想做个…" / "帮我加个功能" / 任何要做新东西的请求** | **`kickoff`（先分类规模、澄清需求、拿批准）** |
| "帮我创建一个项目" / "我想做个 app" | `creation-guide` |
| "我想做个什么？帮我理理需求" | `creation-flow-intent` |
| "帮我把功能拆一下" / "切个 MVP" | `creation-flow-features` |
| "这个功能预算够吗" | `creation-flow-cost` |
| "先出个原型看看" | `creation-flow-prototype` |
| "用什么技术栈" / "定一下方案" | `creation-flow-techspec` |
| "写个开发文档" / "记录一下进度" | `dev-docs` |
| "怎么启动项目" / "加个运行命令" | `project-run` |
| "加个新功能" / "做个需求" | `ui-sync` |
| "ponytail" / "最简单方案" / "别过度设计" | `ponytail` |
| "review 一下有没有过度工程" | `ponytail-review` |
| "审计整个项目的过度工程" | `ponytail-audit` |
| "开始实现" / "照这个方案做" | `subagent-driven-development` |
| "帮我写个实现计划" / "拆一下任务" | `writing-plans` |
| "写这个功能" / "修这个 bug"（写代码前） | `test-driven-development` |
| "这个 bug 反复出现" / "帮我查查为什么" | `systematic-debugging` |
| "做完了" / "修好了"（声称完成前） | `verification-before-completion` |
| "这块写完了，收个尾" / "能合并吗" | `finishing-a-development-branch` |
| "这几个问题一起修" | `dispatching-parallel-agents` |
| "帮我画个架构图" / "对比这两种布局" | `kickoff` → Visual Companion |
| "帮我写个 skill" / "这个 skill 该改改" | `writing-skills` |

> **两层强制触发**：`using-easymint` 规定任何回应或动作前先查 skill（包括澄清性提问和翻代码）；`kickoff` 规定**任何创作性工作前**必须先分类规模、澄清需求并拿到批准。若你发现 Agent 没走流程直接动手，可以直接提醒它先查 skill。

## 架构

```
EasyMint Skill Pack（27 个 skill）
│
├── 【入口层：两层】
│   ├── using-easymint          ← 调度层：先查 skill + Red Flags 表
│   └── kickoff                 ← 创作起手层：三路径分类 + 批准闸门
│       ├── SKILL.md
│       ├── visual-companion.md        (可视化伴侣使用指南)
│       ├── spec-document-reviewer-prompt.md
│       └── scripts/                   (服务端 + 帧模板 + 跨平台启停脚本)
│
├── 【创建期】
│   ├── easymint-core           ← 方法论核心包
│   │   ├── SKILL.md
│   │   ├── references/          (6 个方法论文档)
│   │   └── assets/              (4 个模板 + 74 个品牌规范)
│   ├── creation-guide          ← 创建引导总编排
│   │   ├── SKILL.md
│   │   ├── scenarios.md
│   │   └── cost-map.md
│   └── creation-flow-*         ← 6 个创建引导子阶段
│
├── 【计划期】
│   ├── writing-plans           ← 任务拆解
│   └── executing-plans         ← 同会话批量执行
│
├── 【执行期】
│   ├── subagent-driven-development   ← 执行引擎
│   │   ├── references/               (3 个 prompt 模板)
│   │   └── scripts/                  (3 个辅助脚本)
│   ├── test-driven-development       (+ references)
│   ├── systematic-debugging          (+ references + scripts + assets)
│   ├── verification-before-completion
│   ├── requesting-code-review        (+ references/code-reviewer.md)
│   ├── receiving-code-review
│   ├── using-git-worktrees
│   ├── finishing-a-development-branch
│   └── dispatching-parallel-agents
│
├── 【工程化】
│   ├── dev-docs                ← 文档规范
│   ├── project-run             ← 运行配置
│   └── ui-sync                 ← 状态同步
│
├── 【质量守护】
│   └── ponytail*               ← 3 个反过度工程工具
│
└── 【元技能】
    └── writing-skills          ← 写 skill 即 TDD（+ 3 份重型参考）
```

**依赖关系：**

- `using-easymint` 是全局入口，路由到所有其他 skill
- 创建期：`creation-guide` 路由到 6 个 `creation-flow-*`；`easymint-core` 是方法论核心承载
- 计划 → 执行：`writing-plans` 的产物（方案文件）是 `subagent-driven-development` 的输入
- 执行期内部有明确转入关系（见各 skill 末尾「与其他 skill 的关系」表）
- `verification-before-completion` 是所有 skill 的收口；`finishing-a-development-branch` 以前者的绿测证据为前置
- 贯穿：`ponytail` 三兄弟 + `dev-docs` + `project-run`

## 与同类的区别

| 维度 | EasyMint Skill | superpowers | ai-team-orchestration | multi-agent-orchestration |
|---|---|---|---|---|
| 角色数 | 4（含设计师） | 2（实现者 + 审查者） | 3 | 2（code+ui） |
| 创建引导 | **7 Gate 全流程** | 仅 brainstorming 一环 | 无 | 无 |
| 计划粒度 | 任务清单 + Step | **2-5 分钟 Step** | 无 | 无 |
| 执行期纪律 | **完整**（TDD/调试/验证/收口） | 完整 | 无 | 无 |
| 反合理化机制 | **Red Flags + Rationalizations 全 skill 覆盖** | 全 skill 覆盖 | 无 | 无 |
| 工程化规范 | 文档+任务+运行 全套 | 部分（ledger/brief） | 无 | agent_state.json |
| 质量守护 | Ponytail 三兄弟 | TDD + 两阶段审查 | 无 | 无 |
| 非程序员适配 | **有** | 无（假设对象是工程师） | 无 | 无 |
| 定位 | **全流程方法论（创建期引导 + 执行期纪律）** | 执行期纪律 | 多 Agent 协作 | spec 驱动编排 |


## 安全边界

- **不会**擅自删除用户文件（ponytail 的 "deletion over addition" 仅指代码层面的精简建议）
- **不会**在未确认的情况下执行高风险操作（git push、部署、发版）
- **不会**泄露 API key、token、私人路径到任何公开产物
- **不会**跳过用户确认检查点（原型确认、开发启动、方案选择）
- **不会**在自己主动发起的情况下 `git worktree remove --force`（移除被拒说明有只存在于该工作区的文件，`--force` 会永久销毁它们）
- **不会**在用户未输入精确的 `discard` 一词时丢弃分支
- 所有 Skill 均为 runtime-neutral，不绑定单一 Agent 平台

### 关于随包脚本

本包含 3 个 bash 辅助脚本（`skills/subagent-driven-development/scripts/`）：

| 脚本 | 作用 |
|---|---|
| `sdd-workspace` | 解析并创建工作目录 `.agentskill/sdd/<方案名>/`，写自忽略 `.gitignore` |
| `task-brief` | 从方案文件抽取单个任务全文到独立文件（避免任务文本流经主 Agent 上下文） |
| `review-package` | 生成审查包（commit 列表 + 统计 + 扩展上下文 diff）到文件 |

它们**只做文件读写与 git 查询**，不联网、不下载、不执行外部代码。运行时若无 bash，可按 SKILL.md 中的「文件即接口」约定手工实现等价动作——**脚本是便利设施，不是必需品**。安全审查结论：纯本地文件与 git 操作，无可疑行为。

## 文件结构

```
skills/
├── <skill-name>/
│   ├── SKILL.md          # Skill 主文件（frontmatter + 工作流）
│   ├── references/       # 参考文档（按需加载，只一层深）
│   ├── assets/           # 随包资产（easymint-core：模板与品牌库）
│   ├── scripts/          # 辅助脚本（subagent-driven-development / kickoff）
│   ├── visual-companion.md  # 可视化伴侣指南（kickoff 专用）
│   ├── scenarios.md      # 场景配置（creation-guide 专用）
│   └── cost-map.md       # 成本映射表（creation-guide 专用）
```

## 验证与测试

每个 Skill 建议配合 2-3 个典型测试 prompt 使用。测试 prompt 设计原则：

1. 覆盖最典型的使用场景（happy path）
2. 包含一个稍复杂或有歧义的场景
3. 期望输出简短描述，便于对比

**对纪律类 skill（TDD / 调试 / 验证），压力场景比学术问题更有效**——设计能让 Agent *想*违规的场景（时间压力 + 沉没成本 + 权威压力组合），观察它是否仍守规则。方法见 `writing-skills` 与 `references/testing-skills-with-subagents.md`。

## License

MIT License - 详见 [LICENSE](LICENSE)

## 致谢

本包的方法论来自两条脉络的融合：

**一、创建期引导（本包原生）**

- 7 Gate 创建流程、项目档案组合引擎、非程序员适配、文档四层协议
- 设计资产：74 个品牌设计规范库 + 4 个种子 HTML 模板（来源见 `THIRD_PARTY_NOTICES.md`）

**二、执行期纪律与创作起手（吸收自 Superpowers）**

- [obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent / [Prime Radiant](https://primeradiant.com) — **MIT License**
- 借鉴内容：强制触发层与 Red Flags 表、**创作起手层（三路径分类 + HARD-GATE 批准闸门 + Visual Companion 可视化伴侣）**、两阶段审查、fix loop 与 breaker、ledger 记账、Rulings not stalls、TDD / systematic-debugging / verification-before-completion / worktrees / branch-finishing / code-review 往返 / 并行派发 / writing-skills 元技能，以及全包的 Common Rationalizations 机制
- 完整借鉴清单与改编说明见 [`skills/easymint-core/assets/THIRD_PARTY_NOTICES.md`](skills/easymint-core/assets/THIRD_PARTY_NOTICES.md#25-方法论来源obrasuperpowers)

**其他**

- 多 Agent 编排灵感来自 Karpathy 的 autoresearch
- 反过度工程理念来自 Ponytail 哲学
- 方法论持续迭代，欢迎贡献和反馈

