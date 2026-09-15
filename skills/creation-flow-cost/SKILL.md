---
name: creation-flow-cost
description: >-
  已采集部署/AI/预算、或对话中确认了功能清单，需要校验成本与功能是否冲突、
  冲突时帮用户做取舍时使用（创建项目引导·成本校验）。
---

# 成本校验与取舍

成本信息**默认已在表单采集**（部署方式 / AI 集成 / 预算），作为参考，**无冲突默认跳过，不主动展开成本话题**。仅当功能与成本冲突时才介入。

## 两条入口

- **表单路径**：成本已随表单传给主 Agent（部署「本地/云端/混合」、AI 集成「无/AI 辅助/Agent/多 Agent」、预算「充足/少量/免费」）。主 Agent 在功能共创后对照功能清单校验，无冲突即跳过
- **直接创建路径**：用户跳过表单，成本未采集——在对话引导功能共创时一并确认（问一句预算倾向即可），再按下方冲突规则校验

## 冲突校验（Read cost-map.md 映射表）

按功能清单推导哪些必然产生成本，与用户预算对照。发现矛盾组合主动指出，不装看不见：

- **免费预算 + AI 辅助/Agent/多 Agent** → 冲突（需 API key，按量计费）
- **免费预算 + 云端部署** → 冲突（云服务/服务器有费用）
- **免费预算 + 支付/短信/邮件** → 冲突（必然付费服务）
- 商业场景附成本核算 + 合规提示（隐私/支付/数据）+ 维护说明

## 取舍选项表（冲突时用）

- 不确定项给选项表而非开放问，每项一句话影响，让用户拍板
- 例（AI 对话 + 倾向免费）：A 换免费额度厂商（时效性/限量，适合先体验）/ B 接受成本（轻量每月几块钱）/ C 先砍该功能（以后再加）
- 禁止无脑抛"免费还是付费"空洞选择；免费额度要讲清"有时效性/限量，非长久之计"
- 用户确认的是"取舍"，不是技术选择

---

## 🔴 CHECKPOINT · 成本取舍确认

发现成本冲突时，必须给出 2-3 个具体取舍选项（每项一句话影响），等用户选择后才继续。不能自行决定「换免费方案」或「砍掉功能」。

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| 用户预算信息缺失 | 直接创建路径且用户没说预算 | 问一句「预算倾向？免费/少量/充足」，不展开成本话题——无冲突默认跳过 |
| 多个功能同时冲突 | 功能清单中有 3+ 个成本冲突项 | 按冲突严重度排序，先解决最核心的 1-2 个，其余标注「后续迭代时再评估」 |
| 用户不接受任何取舍 | 坚持免费但要所有付费功能 | 如实说明「这些功能必然产生费用，免费方案无法支持」，给出「先做免费可实现的核心功能」选项 |
| 成本估算不准 | 实际费用与预估偏差大 | 标注「预估值，实际以服务商定价为准」，不编造精确数字；建议用户核实关键服务定价 |

## 反例黑名单（不要做以下事情）

- **不要无冲突时主动展开成本话题**：无冲突默认跳过，不主动说「这个功能可能要花钱」——制造不必要的焦虑
- **不要抛「免费还是付费」空洞选择**：必须给具体选项（换哪个厂商/接受多少成本/砍哪个功能），每项一句话影响
- **不要隐瞒免费额度的限制**：推荐免费额度时必须讲清「有时效性/限量，非长久之计」
- **不要替用户做取舍**：用户确认的是「取舍」，不是技术选择——给出选项让用户拍板
- **不要忽略商业场景的合规成本**：商业交付场景必须附成本核算 + 合规提示（隐私/支付/数据）+ 维护说明

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 预算这种小事，等实现的时候自然会解决
- 用户说「钱不是问题」，那就不用校验成本了
- 现在谈收费用户会觉得我在劝退，先不吭声
- 免费额度很多家都有，差不多够用，不用细说限制
- 这笔钱应该不多，估个数扔上去就行
- 免费 + AI 也不一定非要花钱吧，可能有免费接口
- 无冲突，那就什么都不用提，跳过就行
- 先做出来，等用户真的收到账单再说

**All of these mean: 对照功能清单校验成本冲突，冲突时给出具体取舍选项让用户拍板。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "Budget is a detail we can settle later" | Cost is an architectural constraint, not a detail — it decides which features are even buildable. |
| "The user said money isn't an issue" | "Not an issue" is not "unlimited". Confirm the actual ceiling before designing against it. |
| "Mentioning cost now will scare them off" | Discovering a bill after launch scares them off harder. Quote it up front. |
| "Free tiers are everywhere, that's basically enough" | Free tiers are time-limited and capped by request volume. Say so, or you're selling a dead end. |
| "The amount is small, I'll just estimate something" | A made-up number becomes a broken promise. Mark it as an estimate, cite the provider's pricing. |
| "Free budget plus AI probably has a free option somewhere" | AI integration is metered API usage. There is no free tier that survives real use. |
| "No conflict found, so I can skip the topic entirely" | Correct — no conflict means skip silently. But "skip" means after checking, not instead of checking. |
| "Better to build first and react to the bill" | The bill arrives whether or not the user agreed to it. Conflict found later is rework plus a dispute. |

> 中文说明：默认不主动展开成本话题，但「默认跳过」的前提是**已经校验过**。免费预算撞上 AI/云端/支付/短信必然冲突，必须主动指出并给 2-3 个具体取舍选项（换厂商 / 接受成本 / 砍功能），由用户拍板。
