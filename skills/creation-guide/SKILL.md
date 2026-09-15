---
name: creation-guide
description: >-
  收到创建项目系统消息、需要从模糊想法引导用户到可开发需求时使用（新建项目/
  直接创建场景）。覆盖复杂度判定、场景识别与阶段路由。
---

# 创建项目引导编排

你（主 Agent）收到创建项目系统消息后，按本清单引导。总原则：智能严谨、能繁能简。
系统提示词 <creation_flow> 已给骨架（分工/复杂度判定/两条入口/7 Gate），本清单补细节：复杂度判定 + 场景识别 + 阶段路由。

## 进入引导

- 先用一两句话说明流程地图（想法 → 产品定义 → 原型 → 技术方案 → 开发 → 验证 → 完成，标出当前步）
- 告知用户"想跳过引导可直接自由描述，我会自己理解推进"
- 表单路径：收到结构化信息后**只回复"已确认"**，**不执行任何初始化动作**（不读 skill、不检查环境、不写文档）——初始化在收到 项目初始化信号 系统消息后才开始，见下方「打开新窗口后」
- 直接创建路径：进入本引导

## 复杂度判定（第一优先——先判复杂度，再定流程）

**判断依据是项目实际内容，不是场景、不是表单里的复杂度字段**。项目场景（商业/实际使用/兴趣等）只反映用户意图，不代表项目规模——「做着玩」也可能是个有 UI 的多页面应用，「做成产品」也可能只是个静态单页。**你要综合项目形式、功能清单、功能点数量来判定**，场景仅作参考，可推翻。

| 档位 | 判定标准（满足任一即归入） | 流程走法 |
|------|--------------------------|---------|
| **极简** | 单文件 / 无依赖 / 静态 HTML 单页 / CLI 单命令 | 直接编码，不写文档、不出原型 |
| **简单** | 几个文件、少量依赖、功能点 ≤ 3、无 UI 或极简 UI | 写需求文档 + task.json，**跳过原型** |
| **中等及以上** | 有 UI（多页面/多路由）、多模块、有后端/数据库、功能点 > 3 | **原型前置**（有 UI 时）→ 原型确认 → 完整文档 → 开发 |

补充规则：
- **无 UI 项目**（纯后端 / CLI / API / 库）无论规模都不强制原型——「原型」专指 UI 项目
- 拿不准时**按项目实际内容保守判定**，宁可中等完整，不因想省事降档漏掉原型

## 打开新窗口后：第一句话问原型

收到 项目初始化信号（项目创建完毕）系统消息后（新项目会话的首条回复）：
1. 先按上面「复杂度判定」判断档位
2. **有 UI 且中等及以上** → 第一条消息就问：「是否根据当前的功能需求先产出UI原型？」——说明可以先出原型预览、根据原型调整需求和效果，确认后再完整开发。等用户答复后 Read creation-flow-prototype
3. **简单 / 极简 / 无 UI** → 跳过原型，说明将采用的流程（极简直接编码 / 简单写需求文档+task.json），随后进入对应分支

## 识别场景（Read scenarios.md）

- 从对话信号匹配场景：商业交付 / 实际使用 / 兴趣创作 / 想法验证 / 学习实践 / 技术实验
- 场景影响自动化档位与附加要求（成本/合规/维护）；流程深度由复杂度判定决定（见上「复杂度判定」）
- 用户认知定表达颗粒度（从措辞感知，不询问）
- 认知 ≠ 项目深度：大牛也能做验证原型，新手也能要商用产品

## 按阶段路由（按需 Read 对应子 skill，不一次全读）

- ① 意图采集 → Read creation-flow-intent
- ② 功能共创 → Read creation-flow-features（成本校验内联于本阶段：对照已采集的部署/AI/预算与功能清单，冲突才介入，见 creation-flow-cost）
- ③ 快速原型（**仅中等及以上且有 UI 的项目**）→ Read creation-flow-prototype
- ④ 技术方案+落盘 → Read creation-flow-techspec（G5 含成本确认，与功能共创阶段的校验衔接）
- 每阶段结束过对应 Gate，未过不进入下一步

## 七道 Gate 复核

G1 需求意图 → G2 范围（过大切 MVP）→ G3 原型（**有 UI 且中等以上必经**，简单/极简/无 UI 跳过）→ G4 用户确认原型
→ G5 技术方案（三重验证）→ G6 正式开发 → G7 对照原型验证（G7 在开发后，不在本 skill）

---

## 🔴 CHECKPOINT · 关键决策点

以下节点必须停手等用户确认，不能擅自继续：

1. **复杂度判定结果**：判定为「中等及以上」时，必须先问用户「是否先出 UI 原型？」，用户答复后才进入对应分支
2. **原型确认（G4）**：原型产出后必须等用户明确确认（「可以」「确认」「就这个方向」），不能自行进入开发
3. **技术方案确认（G5）**：技术方案给出后必须等用户确认「方案符合目标和预算」，不能自行落盘 task.json
4. **范围过大时**：需求超出 MVP 范围时，必须先给出 MVP 切分方案让用户确认，不能自行砍掉功能

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| 用户不回复引导问题 | 提问后超过 1 轮无响应 | 给出推荐默认值并标注「按默认推进，可随时调整」，不卡住流程 |
| 复杂度判定模糊 | 项目特征跨档位（如 3 个功能点但有 UI） | 按「中等及以上」保守判定，先问原型；用户明确说「简单做」再降档 |
| 用户需求前后矛盾 | 对话中功能清单与场景/预算冲突 | 主动指出矛盾并给出 2-3 个取舍选项，不自行选择 |
| 原型方向偏离 | 用户看原型后说「不是我想要的」 | 回到需求理解阶段重新采集，不硬改原型；记录偏离原因 |
| 技术方案不可行 | 环境就绪阶段依赖安装/构建失败 | 记录具体问题 → 告知用户 → 让用户决定重试/跳过/手动处理，不静默跳过 |

## 反例黑名单（不要做以下事情）

- **不要跳过复杂度判定直接出原型**：极简/简单项目出原型是过度工程，浪费用户时间
- **不要把场景当复杂度**：「做着玩」也可能是多页面应用，「做成产品」也可能是静态单页——按项目实际内容判定
- **不要连环追问**：一次只问当前前提已定的问题，每题附推荐答案；不开放问答，不一次抛 3 个以上问题
- **不要在表单路径执行初始化**：收到表单结构化信息后只回复「已确认」，不读 skill、不检查环境、不写文档——初始化在项目初始化信号后才开始
- **不要替用户决定技术选型**：技术方案给出推荐和理由，但用户确认的是「方案符合目标和预算」，不是让 AI 自行拍板
- **不要把引导流程走成表单**：引导是对话式的，先给草案让用户改，比让用户从零描述更高效

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 这个项目一看就很简单，不用走复杂度判定
- 用户这么着急，先做个东西出来给他看
- 流程太重了，我自己看着办更快
- 功能点就这么几个，不用数，直接开始写
- 用户既然没说，我按最常见的做法来就行
- 引导问太多会烦，先跳到出原型吧
- 表单都填完了，说明需求已经够了，直接初始化
- 看着玩的项目，做那么细没必要

**All of these mean: 先判复杂度、再定流程，按 Gate 逐道推进。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "This project is obviously simple, no need to assess complexity" | Eyeballing is not assessment. "Making it for fun" can still be a multi-page app — judge by actual content, not vibe. |
| "The user is in a hurry, let me just build something" | Hurry is why the flow exists. Skipping it produces rework, which is slower than the flow. |
| "The process is too heavy for this" | Only the deliverable shrinks with complexity — the gates do not. Downgrade is a decision, not a default. |
| "They haven't said anything, I'll pick the usual approach" | Unstated is not decided. Ask one question with a recommended default. |
| "The form is filled in, so requirements are settled" | Form answers are raw input, not clarified intent. Clarify before routing. |
| "Asking more questions will annoy the user" | Unanswered assumptions annoy them more. One question at a time, each with a recommendation. |
| "I'll ask about the prototype later, after more work" | Prototype comes before docs and dev for UI projects. Later means rebuilding. |
| "Skipping a gate this once won't matter" | The first skipped gate sets the pattern for the whole build. |

> 中文说明：复杂度判定看项目实际内容（形式/功能清单/功能点数量），场景只作参考可推翻；拿不准时按高不按低。Gate 是不可跳过的检查点，跳过第一道就会跳过其余。
