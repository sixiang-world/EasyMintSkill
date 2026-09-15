---
name: finishing-a-development-branch
description: Use when implementation is complete and verification has passed, and you need to decide how to integrate the work - requires detecting the workspace state and presenting the integration menu before taking any action
---

# Finishing a Development Branch

## Overview

**Core principle:** Verify tests → Detect environment → Present options → Execute choice → Clean up.
（核心原则：验证测试 → 检测环境 → 列出选项 → 执行所选 → 清理现场。）

**Violating the letter of this rule is violating the spirit of this rule.**
（违反这条规则的字面要求，就是在违反它的精神。）

这条规则的实质是**决定权归属**：把代码合到哪里、是否开 PR、要不要丢掉——这是人的决定，不是 Agent 的推断。Agent 的职责是把状态查清楚、把选项摆完整、把选中的那件事执行干净。

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."
（开场自报家门：我正在使用 finishing-a-development-branch skill 来完成这次工作。）

## Step 1: Verify Tests

Run the project's full test suite (`npm test` / `cargo test` / `pytest` / `go test ./...`).

**If tests fail**, report the failures and stop — the menu comes after a green suite:

```
Tests failing (<N> failures). Must fix before completing:

[Show failures]
```

**If tests pass:** continue to Step 2.

> 中文对照：跑项目的**全量**测试套件。
>
> **测试失败** → 报告失败、**停下**。菜单只在绿灯之后才端出来：
> 「测试失败（`<N>` 处）。完成之前必须先修：」然后附上失败内容。
>
> **测试通过** → 进入 Step 2。
>
> 为什么先测再问：带着红色的树去问"你想怎么合入"，等于把一个坏选择包装成三个好选项。

## Step 2: Detect Environment

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# Capture now, while still inside the workspace — Step 5 changes directory
# before cleanup (Step 6) needs this value
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

This determines which menu to show and how cleanup works:

| State | Menu | Cleanup |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON` (normal repo) | Standard 3 options | No worktree to clean up |
| `GIT_DIR != GIT_COMMON`, named branch | Standard 3 options | Provenance-based (see Step 6) |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Reduced 2 options (no merge) | Externally managed — leave in place |

> 中文对照：这三条命令决定了后面**端哪份菜单**以及**怎么清理**。
>
> 第三条 `WORKTREE_PATH` 必须在**此时**取——因为 Step 5 会切换目录，而 Step 6 清理时还需要这个值。**先取值、后换目录**，顺序反了就只能凭记忆猜路径。
>
> | 状态 | 菜单 | 清理 |
> |---|---|---|
> | 两值相等（普通仓库） | 标准三选项 | 没有 worktree 需要清理 |
> | 两值不等，且在有名字的分支上 | 标准三选项 | 按归属判断（见 Step 6） |
> | 两值不等，detached HEAD | 精简两选项（**没有合并**） | 由外部管理——原地保留 |
>
> detached HEAD 之所以不能给"本地合并"这个选项：没有分支名可合，合并无处落脚。

## Step 3: Determine Base Branch

The base branch is whatever this work forked from — usually named in the plan, the conversation, or the branch's upstream. If it is not already known, ask: "This branch split from <your best guess> - is that correct?" Confirm before merging: merging into the wrong base is expensive to undo.

> 中文对照：基分支 = 这份工作从哪儿分出来的。线索通常在计划、对话记录，或分支的 upstream 里。**如果还不确定，就直接问**：「这个分支是从 `<你的最佳猜测>` 分出来的，对吗？」——合并之前必须确认。
>
> 为什么必须确认：合错基分支是**贵且难回退**的错误。合进一个错误的分支，撤销要动到已经推上去的历史，而不只是本地一次 reset。一个问句的成本，换一次不可逆操作的正确性。

## Step 4: Present Options

**Normal repo and named-branch worktree — present exactly these 3 options:**

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)

Which option?
```

**Detached HEAD — present exactly these 2 options:**

```
Implementation complete. You're on a detached HEAD (externally managed workspace).

1. Push as new branch and create a Pull Request
2. Keep as-is (I'll handle it later)

Which option?
```

Present the menu exactly as written — concise, with every option coming from the list above. Discarding the work happens only in response to your human partner explicitly asking for it (see "If your human partner asks to discard the work" below). Wait for their answer; the integration decision is theirs.

> 中文对照：
> **普通仓库 + 有名字的分支的 worktree —— 只列这三项：**
> ```
> 实现完成。你想怎么处理？
>
> 1. 本地合并回 <base-branch>
> 2. 推送并创建 Pull Request
> 3. 分支原样保留（我稍后自己处理）
>
> 选哪个？
> ```
>
> **detached HEAD —— 只列这两项：**
> ```
> 实现完成。你当前在 detached HEAD 上（工作区由外部管理）。
>
> 1. 作为新分支推送并创建 Pull Request
> 2. 原样保留（我稍后自己处理）
>
> 选哪个？
> ```
>
> **按原文照端，简短，每一项都来自上面的列表**。"丢弃这份工作"只在用户**明确开口要求**时才会发生（见下文「如果用户要求丢弃这份工作」）。**等他的回答**——合入方式是他的决定。
>
> 不要自作聪明加第 4 项、不要在选项里替用户排序、不要替他勾好默认值。

### 关于创建 PR 的机制

创建合并请求 / PR 时，**使用运行时可用的 PR 机制**——例如 `gh` CLI，或对应平台提供的工具；若都没有，用推送后平台回显的创建链接。同时遵循仓库已有的 PR 模板与约定（若有），最后把 URL 报告给用户。

**不要假定某个特定平台或 CLI 一定存在**：先探测（`gh --version` 之类），有则用，无则走链接方式，并如实说明你用的是哪种。

## Step 5: Execute Choice

### Option 1: Merge Locally

```bash
# Get main repo root for CWD safety
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# Merge first — verify success before removing anything
git checkout <base-branch>
git pull
git merge <feature-branch>

# Verify tests on merged result
<test command>
```

If tests fail on the merged result: stop, leave the worktree and branch in place, and investigate — nothing has been pushed, so the merge is local and recoverable.

Once the merged result is green: clean up the worktree (Step 6), then delete the branch:

```bash
git branch -d <feature-branch>
```

> 中文对照：
> 先求出主仓库根目录并 `cd` 过去（这是为了工作目录安全——删除 worktree 必须在它外面执行）。**先合并、后验证、最后才删除**，顺序不能调换。
>
> **合并结果测试失败** → 停下，**worktree 和分支都原地保留**，去排查。理由：还没有任何东西被推上去，这次合并是本地可回退的——保留现场，损失为零；先删再查，就把可回退变成不可回退。
>
> **合并结果绿灯** → 清理 worktree（Step 6），然后删分支。
> 注意删分支用的是 `-d`（小写，安全删除）——**不允许在未合并的状态下用它**，这是 git 自带的最后一道防线。

### Option 2: Push and Create PR

```bash
git push -u origin <feature-branch>
# From a detached HEAD, name the new branch on the remote:
# git push origin HEAD:refs/heads/<new-branch>
```

Then create the pull/merge request against <base-branch> with the runtime's available PR mechanism — `gh` CLI or the corresponding platform tool if one is available, or the creation URL most forges print when you push — following the repo's PR template and conventions if present, and report the URL to your human partner.

**Keep the worktree** — your human partner iterates on PR feedback there.

> 中文对照：先把分支推上去（detached HEAD 时用后面的写法在远端**命名一个新分支**）。
>
> 然后用**运行时可用的 PR 机制**创建针对 `<base-branch>` 的合并请求——有 `gh` CLI 或对应平台工具就用它，没有就用推送后平台回显的创建链接。有 PR 模板和仓库约定就遵循，最后**把 URL 报告给用户**。
>
> **worktree 必须保留**——用户会在那里迭代 PR 反馈。PR 开出来不是终点，评论和修改还要落在这个工作区里，提前清掉等于把用户的工作台拆了。

### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."
（报告：「保留分支 `<name>`。worktree 保留在 `<path>`。」）

### If your human partner asks to discard the work

This path exists only as a response to an explicit request to throw the work away. Confirm first:

```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for that exact confirmation. When it arrives:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

Then clean up the worktree (Step 6) and force-delete the branch:

```bash
git branch -D <feature-branch>
```

> 中文对照：这条路径**只在用户明确要求把工作丢掉时**才存在。**必须先确认**：
>
> ```
> 这会永久删除：
> - 分支 <name>
> - 全部提交：<commit-list>
> - <path> 处的 worktree
>
> 输入 'discard' 确认。
> ```
>
> **等待这个精确的确认词。** 收到之后：先切到主仓库根目录，再清理 worktree（Step 6），最后强删分支（`git branch -D`，大写 D）。
>
> **为什么要求用户亲手打出 `discard` 这个词**：这是不可逆操作的最后一道心理闸门。用户在浏览菜单时顺口说"行，把它弄掉吧"是**随口应答**，不是对"永久删除这些具体提交"这个事实的确认。要求他打出这个词，就是强制这个事实被完整读一遍。**别把任何一句话当作 `discard`。**

## Step 6: Cleanup Workspace

**Runs for Option 1 and confirmed discards.** Options 2 and 3 always preserve the worktree. Both callers have already changed directory to the main repo root — worktree removal must run from outside the worktree — and use the `GIT_DIR`/`GIT_COMMON`/`WORKTREE_PATH` values captured in Step 2, from before that directory change.

**If `GIT_DIR == GIT_COMMON`:** Normal repo, no worktree to clean up. Done.

**If `WORKTREE_PATH` is under `.worktrees/` or `worktrees/`:** We created this worktree — we own cleanup:

```bash
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

**If removal is refused** (`contains modified or untracked files`): the worktree holds files that exist nowhere else — uncommitted plans, notes, or scratch work. Never `--force` on your own initiative. Show your human partner what is at stake and ask:

```bash
git -C "$WORKTREE_PATH" status --porcelain -uall
```

```
Worktree removal refused — these files were never committed:

<file list>

1. Commit them to <branch> before cleanup
2. Move them into <main repo root>
3. Delete them (unrecoverable)

Which?
```

Carry out the choice, then remove the worktree.

**Otherwise:** The host environment owns this workspace — leave it in place. If your platform provides a workspace-exit tool, use it.

> 中文对照：
> **只有选项 1 和"确认丢弃"会走到这里。** 选项 2 和 3 **永远保留 worktree**。两个调用方都已经先切到主仓库根目录——**删除 worktree 必须在它外面执行**——并使用 Step 2 在换目录**之前**取到的 `GIT_DIR` / `GIT_COMMON` / `WORKTREE_PATH`。
>
> **两值相等（普通仓库）** → 没有 worktree 要清理，结束。
>
> **`WORKTREE_PATH` 位于 `.worktrees/` 或 `worktrees/` 下** → 这个 worktree 是我们建的，清理归我们：移除它，再 `git worktree prune` 自愈式清掉残留的注册记录。
>
> **移除被拒**（提示 `contains modified or untracked files`）→ 说明这个 worktree 里有**只存在于此处**的文件：没提交的计划、笔记、草稿。**绝不主动加 `--force`。** 先跑 `git status --porcelain -uall` 把风险摊开给用户看，然后让他在三条路里选：
> ```
> worktree 移除被拒 —— 这些文件从未被提交过：
>
> <文件列表>
>
> 1. 先把它们提交到 <branch>，再清理
> 2. 把它们移到 <主仓库根目录>
> 3. 删除它们（不可恢复）
>
> 选哪个？
> ```
> 执行所选，然后再移除 worktree。
>
> **其他情况** → 这个工作区归宿主环境所有，**原地保留**。若你的平台提供了退出工作区的工具，用它。
>
> 为什么 `--force` 一个字都不能自己加：移除被拒的信号含义是"这里有别处不存在的文件"。`--force` 正是绕过这道保护，把这些文件永久销毁。**保护机制给出的拒绝，不是需要你去绕过的障碍。**

## Quick Reference

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | yes | - | - | yes |
| 2. Create PR | - | yes | yes | - |
| 3. Keep as-is | - | - | yes | - |
| Discard (explicit request only) | - | - | - | yes (force) |

> 中文对照：
> | 选项 | 合并 | 推送 | 保留 worktree | 清理分支 |
> |---|---|---|---|---|
> | 1. 本地合并 | 是 | - | - | 是 |
> | 2. 创建 PR | - | 是 | 是 | - |
> | 3. 原样保留 | - | - | 是 | - |
> | 丢弃（仅在明确要求时） | - | - | - | 是（强删） |
>
> 读表时注意两处：**选项 2 不删分支、不删 worktree**（PR 反馈要在那里改）；**丢弃是唯一出现 `-` 合并 + 强删分支的一行**，也是唯一需要用户亲手打字的路径。

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Tests passed earlier this session" | Run the suite on the tree you are about to integrate. A green run only proves the tree it ran on. |
| "They obviously want it merged" | Integration is your human partner's decision. Present the menu and wait. |
| "They seem done with this feature — I'll offer to discard it" | The menu is complete as written. Discard happens only when your human partner asks for it in so many words. |
| "'Yeah, get rid of it' counts as confirmation" | Only the typed word `discard` authorizes deletion. |
| "The PR is up, so the worktree is clutter now" | PR feedback gets fixed in that worktree. It stays until the work lands. |
| "This other worktree looks stale — I'll clean it too" | Clean up only worktrees under `.worktrees/` or `worktrees/`. Everything else belongs to the host. |
| "Removal refused — `--force` is just finishing the cleanup" | The refusal means files exist only in that worktree. `--force` destroys them permanently. Show your human partner and ask. |
| "The merged-result failure is probably flaky" | A failing merged result stops everything. Branch and worktree stay put while you investigate. |
| "The base branch is obviously main" | Confirm the fork point or ask. Merging into the wrong base is expensive to undo. |
| "The push was rejected — force-push will fix it" | A rejected push means the remote moved. Investigate; force-push only on your human partner's explicit request. |

> 中文对照：
> - **"这次会话早些时候测试是过的"** → 在你**即将合入的那棵树**上重跑一遍套件。一次绿灯只证明它当时跑的那棵树。
> - **"他们显然想合并"** → 合入是用户的决定。端出菜单，然后等。
> - **"他们这个功能看起来做完了，我顺便提供个丢弃选项吧"** → 菜单按原文就是完整的。丢弃只在用户**明说了**的时候发生。
> - **"'行，弄掉吧' 算确认"** → 只有亲手打出的 `discard` 这个词才授权删除。
> - **"PR 都开了，worktree 现在是累赘"** → PR 反馈要在这个 worktree 里改。它留到工作真正落地为止。
> - **"另一个 worktree 看着像废弃的，我一并清了"** → 只清理 `.worktrees/` 或 `worktrees/` 下的 worktree。其他一切归宿主所有。
> - **"移除被拒——`--force` 只是把清理做完"** → 被拒意味着有文件只存在于那个 worktree 里。`--force` 会永久销毁它们。摊给用户看，然后问。
> - **"合并结果失败大概是 flaky 吧"** → 合并结果失败要**停掉一切**。分支和 worktree 原地不动，去排查。
> - **"基分支显然就是 main"** → 确认分叉点，或者直接问。合错基分支代价很高、很难回退。
> - **"推送被拒绝了——强推一下就解决"** → 推送被拒说明远端已经动了。去排查；强推只在用户明确要求时才做。

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 实现刚写完，还没验证过 | `verification-before-completion`——**先过它的闸门**，本 skill 的 Step 1 才有绿灯可验 |
| 测试是红的 | 停下本 skill；先修。根因不明转 `systematic-debugging` |
| 需要补测试才能绿 | `test-driven-development` |
| 合并结果测试失败 | 停下；worktree 与分支原地保留，回到 `systematic-debugging` |
| 还没有隔离工作区就已开工 | `using-git-worktrees`——本 skill 的 Step 2 / Step 6 依赖它建立的现场 |
| 被拒的推送、合并冲突等操作异常 | 按情况转 `systematic-debugging`；不要靠强推 / 强删绕过 |

**本 skill 是流程的最后一环**：它的前置是 `verification-before-completion` 给出的绿测证据，它的现场是 `using-git-worktrees` 建立的隔离工作区。**三者缺一，"完成"就只是状态描述，不是事实。**
