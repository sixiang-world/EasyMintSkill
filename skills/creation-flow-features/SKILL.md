---
name: creation-flow-features
description: >-
  创建项目引导·功能共创。需求意图已明确、需要细化功能范围、把大想法切 MVP 时使用。
---

# 功能共创

你（主 Agent）在创建项目引导的第二步，帮用户把意图细化为可开发的功能清单。

## 能繁能简

- 用户能补充细节就聊，不能就代填草案让用户挑
- 想法过大 → 切 MVP：完整愿景 → 核心价值 → 最小可验证功能 → MVP
- 不拒绝，而是帮用户把愿景变成能开始的项目

## 需求清单

- 按用户故事 P1/P2/P3 排序（P1 = MVP 必须，P3 = 以后再说）——每个故事可独立测试
- 呈现确认 → 得到用户认可的功能清单（**此阶段不落盘文档**）——需求清单是后续原型的依据，先锁"做什么/为什么"；技术（How）留到技术方案阶段
- 完整项目文档（需求文档 + 技术架构 + task.json）在原型确认后才写，见 creation-flow-prototype

## 成本校验（内联于本阶段，冲突才介入）

功能清单确定后，对照已采集的部署/AI/预算（或直接创建时用户确认的预算）校验成本冲突——无冲突默认跳过，不主动展开；发现冲突（如免费预算 + AI 辅助/Agent/云端）主动指出并给取舍选项，见 creation-flow-cost。

---

## 🔴 CHECKPOINT · 功能清单确认

功能清单呈现后必须等用户明确确认（「可以」「就这些」「确认」），才能进入下一阶段。用户修改后需重新呈现更新版清单，再次确认。

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| 用户功能清单无限增长 | 每轮都加新功能，P1 超过 10 个 | 主动提醒「P1 已超过 10 个，建议切 MVP——哪些是没它就不成立的核心功能？」，帮用户聚焦 |
| 用户拒绝切 MVP | 坚持「所有功能都要做」 | 给出两版对比：MVP 版（可立即开始）vs 完整版（需分阶段），让用户选；不自行砍掉功能 |
| 功能描述模糊 | 用户说「加个用户系统」但不说具体能力 | 拆解为具体用户故事（注册/登录/权限/个人中心），标注「⚠️ 待确认」，让用户逐项确认 |
| 功能间冲突 | 两个功能需求互斥（如「完全离线」+「云端同步」） | 主动指出冲突，给出取舍选项，不自行选择 |

## 反例黑名单（不要做以下事情）

- **不要在此阶段落盘文档**：需求清单是对话中的确认，不写 docs/需求文档.md——完整文档在原型确认后才写
- **不要讨论技术实现**：功能共创只锁「做什么/为什么」，不讨论「用什么技术/怎么实现」——技术留到技术方案阶段
- **不要把 P3 功能混进 MVP**：P1 = MVP 必须，P2 = 后续迭代，P3 = 以后再说——排序后明确标注，不模糊处理
- **不要拒绝用户的大想法**：用户说「我想做个平台级应用」时，不说「太大了做不了」，而是帮用户从愿景切到 MVP
- **不要跳过成本校验**：功能清单确定后必须对照预算检查冲突，无冲突才跳过——发现冲突必须主动指出

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 这些功能都不难，一次全做了吧，省得以后返工
- 用户想全都要，那就别扫兴，先列进去
- MVP 听起来像偷工减料，直接做完整版更专业
- P1 已经有十二个了，但每个都挺重要的
- 用户又加了个功能，加上去就好，不用提醒他
- 切 MVP 用户可能不高兴，先不切，做到哪算哪
- 功能他描述得模糊，我按自己的理解补全就行
- 成本校验等做完再说，现在谈钱太早了

**All of these mean: 切 MVP、明确 P1/P2/P3 排序，并对照预算校验成本冲突。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "None of these are hard, let's do them all at once" | "Not hard" individually is not "not hard" together. Scope is what kills projects, not per-feature difficulty. |
| "The user wants everything, don't disappoint them" | Refusing to cut is what actually disappoints them — with an unfinished project. Cut, show both versions, let them choose. |
| "MVP sounds like doing a half-assed job" | MVP is the fastest path to a working product. The full version ships later, from a real base. |
| "Every P1 feels essential" | If 12 items are all P1, then P1 means nothing. Ask which ones the product cannot exist without. |
| "They added one more feature — just add it quietly" | Silent acceptance is how P1 crosses 10. Flag the growth explicitly every time it happens. |
| "Cutting scope will upset them" | They get upset by rework, not by a clear MVP/complete comparison. Show both. |
| "Their description is vague, I'll interpret it" | A vague feature becomes a vague build. Decompose to concrete user stories, mark ⚠️ 待确认, get sign-off. |
| "Cost checks can wait until we build" | Cost conflict discovered after the feature list is locked is a redesign, not a check. Inline it now. |

> 中文说明：范围膨胀靠「主动提醒 P1 超过 10 个」拦截，不靠用户自觉。切 MVP 是给对比让用户选，不是替用户砍功能；成本冲突必须在清单锁定前指出。
