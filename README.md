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

## Skill 清单

| 分类 | Skill | 用途 |
|---|---|---|
| **核心** | `easymint-core` | 方法论核心包：主 Agent 系统提示词 + 13 条规则 + 4 Agent 模板 + 多 Agent 编排 + 7 Gate 流程 + 工程化机制 + 文档协议 |
| **创建引导** | `creation-guide` | 创建项目总编排：复杂度判定 + 场景识别 + 阶段路由 + 7 Gate 复核 |
| | `creation-flow-intent` | 意图采集：从模糊想法到具体目标，前沿轮次制 |
| | `creation-flow-features` | 功能共创：细化功能范围，P1/P2/P3 排序，切 MVP |
| | `creation-flow-cost` | 成本校验：部署/AI/预算与功能冲突检测，取舍选项 |
| | `creation-flow-prototype` | 快速原型：HTML 原型产出，渲染正确性审查，分档修改 |
| | `creation-flow-techspec` | 技术方案：三重验证 + 环境就绪 + 落盘 task.json |
| **工程化** | `dev-docs` | 项目文档规范：导航页 / 日期明细 / CHANGELOG / 技术架构四层 |
| | `project-run` | 运行配置：`run.json` 格式规范 + 脚本管理 |
| | `ui-sync` | 新需求任务状态同步：task.json 状态与运行时进度同步 |
| **质量守护** | `ponytail` | 反过度工程主规则：最懒可行方案，YAGNI，stdlib 优先 |
| | `ponytail-review` | 过度工程 Code Review：diff 级别的复杂度审计 |
| | `ponytail-audit` | 全仓过度工程审计：ranked list of what to delete/simplify |

## 快速开始

### 安装到 Claude Code

```bash
# 方式一：一键安装（推荐）
npx skills add EasyMintSkill

# 方式二：手动复制
git clone https://github.com/sixiang-world/EasyMintSkill.git
cp -r EasyMintSkill/skills/* ~/.claude/skills/
```

### 安装到其他 Agent Runtime

EasyMint Skill 兼容所有支持 Agent Skills 标准的 runtime（Claude Code、Codex、Cursor、OpenClaw、Hermes 等）。将 `skills/` 目录下的任意 Skill 复制到对应 runtime 的 skills 目录即可。

### 触发方式

| 你想说的话 | 触发的 Skill |
|---|---|
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

## 架构

```
EasyMint Skill Pack
├── easymint-core          ← 方法论核心包
│   ├── SKILL.md
│   └── references/         (6 个方法论文档)
├── creation-guide          ← 创建引导总编排
│   ├── SKILL.md
│   ├── scenarios.md
│   └── cost-map.md
├── creation-flow-*         ← 6 个创建引导子阶段
├── dev-docs                ← 文档规范
├── project-run             ← 运行配置
├── ui-sync                 ← UI 状态同步
└── ponytail*               ← 3 个反过度工程工具
```

**依赖关系：** `creation-guide` 路由到 6 个 `creation-flow-*` 子 Skill；`easymint-core` 是方法论核心承载，其他 Skill 是其具体落地。

## 与同类的区别

| 维度 | EasyMint Skill | ai-team-orchestration | multi-agent-orchestration |
|---|---|---|---|
| 角色数 | 4（含设计师） | 3 | 2（code+ui） |
| 创建引导 | 7 Gate 全流程 | 无 | 无 |
| 工程化规范 | 文档+task+run 全套 | 无 | agent_state.json |
| 质量守护 | Ponytail 三兄弟 | 无 | 无 |
| 定位 | 全流程开发方法论 | 多 Agent 协作 | spec 驱动编排 |

## 安全边界

- **不会**擅自删除用户文件（ponytail 的 "deletion over addition" 仅指代码层面的精简建议）
- **不会**在未确认的情况下执行高风险操作（git push、部署、发版）
- **不会**泄露 API key、token、私人路径到任何公开产物
- **不会**跳过用户确认检查点（原型确认、开发启动、方案选择）
- 所有 Skill 均为 runtime-neutral，不绑定单一 Agent 平台

## 文件结构

```
skills/
├── <skill-name>/
│   ├── SKILL.md          # Skill 主文件（frontmatter + 工作流）
│   ├── references/       # 参考文档（按需加载）
│   ├── scenarios.md      # 场景配置（creation-guide 专用）
│   └── cost-map.md       # 成本映射表（creation-guide 专用）
```

## 验证与测试

每个 Skill 建议配合 2-3 个典型测试 prompt 使用。测试 prompt 设计原则：
1. 覆盖最典型的使用场景（happy path）
2. 包含一个稍复杂或有歧义的场景
3. 期望输出简短描述，便于对比

## License

MIT License - 详见 [LICENSE](LICENSE)

## 致谢

- 多 Agent 编排灵感来自 Karpathy 的 autoresearch
- 反过度工程理念来自 Ponytail 哲学
- 方法论持续迭代，欢迎贡献和反馈
