---
name: dev-docs
description: >-
  项目文档记录规范。开发期维护 docs/开发记录.md（导航页）、docs/开发记录/（按日期明细）、
  CHANGELOG.md（正式发布日志）、docs/技术架构.md（架构设计）时使用。
  提供进度快照、会话交接、按日期分文件、发版记录、架构维护的完整规则。
---

# Dev Docs — 项目文档记录规范

开发期维护项目文档时按本规范执行。极简项目（单文件/无依赖）不建文档体系；有文档体系的项目按本规范执行。

## 文档分层

```
docs/开发记录.md          导航页（方法论核心承载）：头部「当前进度快照」+「开发记录索引」
docs/开发记录/<日期>.md   按日期分文件的进度明细（只增不改，带提交 hash）
CHANGELOG.md              正式发布日志（Keep a Changelog，只记用户可见变更）
docs/技术架构.md          架构设计（只追加，记录技术选型与关键决策）
```

分工：**导航页管「现在在哪」，明细管「每天发生了什么」，CHANGELOG 管「发布了什么」，技术架构管「系统怎么设计」**——四者不重复记录同一内容。

## docs/开发记录.md（导航页）

本文件是项目进度与交接的**方法论核心承载**，会话交接时更新。两部分：

### 头部「当前进度快照」

记录当前状态，**每次会话交接时更新**（本区可改）：

```
### 当前版本
- **vX.Y.Z**（日期发布：一句话变更摘要）

### 最近工作
**日期 变更主题**（详见 docs/开发记录/<日期>.md）：
- 本次变更要点（2-4 条，可追溯明细）

### 接下来安排
1. 未完成的待办（按优先级）
```

- 「最近工作」保留最近 1-2 次会话即可，更早的折叠到下方时间线或由明细承接
- 更新时机：**会话结束时**（交接给下一次会话）、每次发版后
- 新会话开始时**先读本文件头部快照**，了解当前状态再开工

### 开发记录索引

日期 → 日志文件映射（按日期降序），每天日志独立存 `docs/开发记录/<日期>.md`——**按需读取，避免单文件无限膨胀**：

```
| 日期 | 记录 |
|------|------|
| 2026-08-19 | 27. 移除请求超时 & 会话卡死修复（2026-08-19，v0.10.2） |
```

- 每次新建当天日志时，在索引表加一行（编号递增）
- 同一天多条记录合并为一行（用分号分隔）

## docs/开发记录/<日期>.md（按日期明细）

- 文件名如 `2026-08-19.md`，记录当天**项目变动 / 用户决策 / 实现详录**
- **只增不改**：当天记录追加到当天文件，不覆盖历史；已写内容不修改
- 每条记录带**日期 + 提交 hash 关联**，方便回溯（如「2026-08-19 移除请求超时（commit c51e9e2）」）
- 记录内容：用户需求与决策、实现方案要点、踩坑与根因、发布记录
- 写入时机：每个任务完成、每个阶段切换时追加

## CHANGELOG.md（正式发布日志）

遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/) 规范，**只记录 release 的用户可见变更**：

- **日常变更先记入 `[Unreleased]` 区块**，发版时整理成版本条目
- 版本条目格式：`## vX.Y.Z (日期) — 变更主题`，下分 `### Added / Changed / Fixed / Removed`（按需）
- 只写**用户能感知**的变更（功能/修复/交互/性能），不写内部实现细节
- 与 `docs/开发记录.md` 分工：CHANGELOG 面向用户看版本变化，开发记录面向开发看过程
- 发版流程：① 更新版本号 + 整理 CHANGELOG 条目 → ② 打 tag → ③ 推送触发 CI 自动 Release

## docs/技术架构.md（架构设计）

记录系统架构与关键决策，**只追加、不重写**：

- 记录内容：系统架构、数据模型、模块划分、关键设计决策（含**为什么这么选**）
- 每次架构级变更追加一个新章节（如「设备互联（v0.6.6）」「会话隔离（v0.6.5）」），标注版本
- **不记录**：代码能自解释的细节、临时方案、一次性决策（这些进开发记录明细）
- 发现过时描述（如已删除的机制仍被引用）时**主动修正**，避免误导
- 新会话涉及架构问题时**先读本文件**再动手

## 通用规则

- 文档用中文命名，放入 `docs/` 目录
- **增量更新优先**：追加新内容直接写；修改或删除已有内容，先向用户列出要改的部分确认
- 写文档时**所有占位符（{{PROJECT_NAME}}、[待填写]）必须替换为实际内容**，禁止留空
- 归档：定稿且不再维护的方案文档移入 `docs/archive/`，当前文档区只保留有效文档

---

## 🔴 CHECKPOINT · 文档修改确认

- **修改或删除已有内容**：必须先向用户列出要改的部分确认，不能自行修改历史记录
- **会话交接时**：必须更新 `docs/开发记录.md` 头部快照，不能只写明细不更新导航
- **发版时**：必须整理 CHANGELOG 条目 + 打 tag，不能只写开发记录不更新正式日志

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| 文档文件不存在 | docs/目录或开发记录.md 未创建 | 先创建目录结构和导航页模板，再写入内容；不跳过文档体系直接编码 |
| 占位符未替换 | 生成的文档含 {{PROJECT_NAME}} 或 [待填写] | 立即替换为实际内容；无法确定的标注「⚠️ 待确认」并告知用户，不留空 |
| 索引与明细不一致 | 开发记录索引表缺少某天的记录 | 补全索引行；以明细文件为准，索引是导航不是真相源 |
| 文档过度膨胀 | 单个明细文件超过 500 行 | 按主题拆分或归档早期内容；不把所有历史堆在一个文件里 |

## 反例黑名单（不要做以下事情）

- **不要为极简项目建文档体系**：单文件/无依赖的极简项目不建 docs/，不写开发记录
- **不要修改历史明细**：`docs/开发记录/<日期>.md` 只增不改，已写内容不修改——错误用追加更正说明
- **不要把内部实现写进 CHANGELOG**：CHANGELOG 只记用户可见变更，不写机制/实现细节/内部规则
- **不要留占位符**：生成文档时所有 {{PROJECT_NAME}}、[待填写] 必须替换为实际内容，禁止留空
- **不要跳过会话交接更新**：每次会话结束必须更新开发记录头部快照，新会话开始先读快照

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 先把功能做完，文档最后统一补一份
- 这次改动太小了，不值得记进开发记录
- 模板里的占位符用户一看就懂，回头他会自己填
- CHANGELOG 和开发记录差不多，写一个就够了
- 会话要结束了，快照下次会话再更新也一样
- 历史记录里有条写错了，顺手改掉更干净
- 架构没大变，技术架构文档不用动
- 文档要写这么多字，占用了开发时间

**All of these mean: 任务完成即追加当天明细 + 更新导航页快照，占位符全部替换为实际内容。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "Finish the features first, write all the docs at the end" | Retrospective docs are reconstructed from memory — they lose the decisions and the root causes, which are the only parts worth keeping. |
| "This change is too small to record" | Small changes are exactly what the daily log exists for. The next session cannot infer them from the diff. |
| "The placeholder is obvious, they'll fill it in" | A placeholder that ships is a broken document. Replace it or mark ⚠️ 待确认 and say so. |
| "CHANGELOG and the dev log say the same thing, one is enough" | Different readers: CHANGELOG is user-visible changes per release, the dev log is process per day. Merging loses both. |
| "I'll refresh the snapshot next session" | Next session starts by reading that snapshot. A stale snapshot is worse than none — it misleads. |
| "There's an error in the history, I'll just correct it" | Daily details are append-only. Correct with a new appended note so the record stays traceable. |
| "The architecture didn't really change, skip the doc" | Architecture-level decisions are what nobody can recover from code later. If you decided it, append it. |
| "Writing docs eats into build time" | Docs are the handoff medium. Without them, every session restarts from zero — that is the real cost. |

> 中文说明：文档是给下一个会话（或下一个人）看的交接物，不是给自己看的总结——所以「以后补」等于丢失。明细只增不改，改历史要用追加更正说明。极简项目（单文件/无依赖）不建文档体系，这条例外只适用于极简。
