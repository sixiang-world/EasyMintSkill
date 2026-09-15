---
name: creation-flow-intent
description: >-
  创建项目引导·意图采集。用户刚创建项目、还不知道想做什么、需要从模糊想法
  引出具体目标时使用。
---

# 意图采集

你（主 Agent）在创建项目引导的第一步，帮用户从模糊想法落到具体目标。

## 开场

- 一句开场："想做个什么？一句话说清即可"——一次只问一个，不连环追问
- 用户说想法 → 先形成理解草案呈现让用户改（先给草案、用户确认或修改，比让用户从零描述更高效）
- 没想法 → 灵感清单（产品方向而非技术选项）：记账/二手交易/兴趣社区/学习打卡/宠物助手…

## 前沿轮次制

- 把需求看成决策树：每轮只问当前前提已定的问题（前沿），一次给一轮
- 每题附推荐答案/默认选项，用户选改即可——不开放问答，不连环追问
- 事实自查：技术事实/行情/外部服务能力用网页抓取查，不抛给用户
- 依赖未决的问题自动延后到下一轮
- 完工判据 = "前沿清空"（没有前提已定的可答问题了），不是"问了 N 个问题"

---

## 🔴 CHECKPOINT · 意图确认

意图采集完成后（前沿清空），必须用一句话复述理解草案：「我理解你想做的是 [核心目标]，面向 [目标用户]，解决 [核心问题]。对吗？」——等用户确认或修改后，才进入功能共创阶段。

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| 用户说「随便」「都行」 | 意图完全模糊 | 给 3 个具体方向选项（附一句话说明），让用户挑一个，不开放问答 |
| 用户想法过大 | 一句话描述涉及 5+ 功能模块 | 先肯定愿景，然后问「最想先验证哪个核心价值？」——切到 MVP 思维，不试图一次采集全部 |
| 用户描述技术方案而非需求 | 「我想用 React 做个...」 | 先记技术偏好，然后追问「用它来解决什么问题？谁会用？」——需求优先于技术 |
| 多轮后仍模糊 | 3 轮后前沿仍未清空 | 给出综合理解草案让用户改，标注「这是我基于对话的理解，你直接改不对的地方」，不继续追问 |

## 反例黑名单（不要做以下事情）

- **不要问技术选项**：意图采集阶段不问「用 React 还是 Vue」「前端还是全栈」——技术留到技术方案阶段
- **不要连环追问**：一次只问一个问题，不抛「你想做什么？给谁用？解决什么问题？有什么特色？」连珠炮
- **不要给技术灵感清单**：没想法时给产品方向（记账/社区/打卡），不给技术方向（做个 CLI/写个 API）
- **不要跳过理解草案**：用户说完想法后必须先给草案让用户改，不能直接进入功能清单
- **不要把「没想法」当失败**：用户说「不知道做什么」是正常状态，给灵感清单即可，不催促

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 用户给的信息已经够多了，可以开始做功能清单了
- 再问下去用户会烦，先建个草案往下走
- 意图这么模糊，我替用户想一个合理的就行
- 用户说「随便」「都行」，那就按最常规的理解来
- 他提到 React，那就是想做前端应用，需求已经明确了
- 已经问了两轮，前沿肯定清空了
- 用户催得急，澄清留到后面遇到问题再说
- 这个想法太大了，先不指出，做着做着就清楚了

**All of these mean: 继续采集，直到「前沿清空」并复述理解草案等用户确认。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "I have enough to start the feature list" | Enough for a guess, not for a build. Frontline empty is the completion test, not "I feel informed." |
| "More questions will annoy the user" | Each question carries a recommended default — that is guidance, not interrogation. One at a time. |
| "Their intent is vague, so I'll pick a reasonable interpretation" | A reasonable interpretation you never confirmed is an assumption you will pay for twice. Draft it, show it, let them edit. |
| "They said 'whatever', so I'll take the standard reading" | "Whatever" means they need options, not silence. Offer 2–3 concrete directions. |
| "They mentioned React, so the requirement is clear" | Technology is a preference, not a requirement. Ask what problem it solves and who uses it. |
| "Two rounds in, the frontline must be clear" | Frontline is clear when no answerable question has a settled premise — count that, not your rounds. |
| "The user is rushing, I can clarify later" | Clarifying later means re-clarifying after the build went the wrong way. |
| "The idea is big — I'll just start and let it narrow" | Big ideas narrow into MVP only when someone asks which core value comes first. That someone is you. |

> 中文说明：完工判据是「前沿清空」，不是「问了几轮」；事实性问题自己查（网页抓取），不抛给用户。意图模糊时给具体选项，不给开放问答。用户没说清细节是常态，不是可以替他决定的理由。
