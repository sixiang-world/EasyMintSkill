---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from the current workspace, or before executing an implementation plan - ensures an isolated workspace exists before edits begin
---

# Using Git Worktrees

## Overview

**Core principle:** Detect existing isolation first. Then use native tools. Then fall back to git. Never fight the harness.
（核心原则：先检测是否已经隔离；有原生机制就用原生；都没有才退回 git。永远不要和运行时对抗。）

隔离工作区的目的是保护当前分支不被半成品污染，也让"实验失败"的代价降到一个目录。但它有一个前提：**这个隔离必须是运行时看得见的**。绕过运行时自带的隔离机制去手工建目录，你得到的不是隔离，而是运行时无法感知、无法清理、无法正确提示的幻影状态。

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."
（开场自报家门：我正在使用 using-git-worktrees skill 来搭建隔离工作区。）

## Step 0: Detect Existing Isolation

**Before creating anything, check if you are already in an isolated workspace.**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**Submodule guard:** `GIT_DIR != GIT_COMMON` is also true inside git submodules. Before concluding "already in a worktree," verify you are not in a submodule:

```bash
# If this returns a path, you're in a submodule, not a worktree — treat as normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

> 中文对照：
> **先检测，再动手**——在创建任何东西之前，先确认自己是不是已经处在一个隔离工作区里。
>
> 三条命令依次取出：git 目录的物理路径、git 公共目录的物理路径、当前分支名。
>
> **submodule 陷阱**：在 git 子模块内部，`GIT_DIR != GIT_COMMON` **同样成立**。所以看到两者不等时，不能立刻下结论说"已经在 worktree 里"，必须先排除子模块——第二条命令若返回了路径，说明你在子模块里，不是 worktree，按普通仓库处理。

**If `GIT_DIR != GIT_COMMON` (and not a submodule):** You are already in a linked worktree. Skip to Step 2 (Project Setup). Do NOT create another worktree.

Report with branch state:
- On a branch: "Already in isolated workspace at `<path>` on branch `<name>`."
- Detached HEAD: "Already in isolated workspace at `<path>` (detached HEAD, externally managed). Branch creation needed at finish time."

> 中文对照：如果两值不等且不在子模块中，说明已经在关联 worktree 里了 → **直接跳到 Step 2（项目初始化），不要再建一个 worktree**（嵌套 worktree 是纯粹的自我伤害）。
>
> 报告时带上分支状态：
> - 在某个分支上：「已在隔离工作区 `<path>`，分支 `<name>`。」
> - detached HEAD：「已在隔离工作区 `<path>`（detached HEAD，由外部管理）。收尾时需要创建分支。」

**If `GIT_DIR == GIT_COMMON` (or in a submodule):** You are in a normal repo checkout.

Has the user already indicated their worktree preference in your instructions? If not, ask for consent before creating a worktree:

> "Would you like me to set up an isolated worktree? It protects your current branch from changes."

Honor any existing declared preference without asking. If the user declines consent, work in place and skip to Step 2.

> 中文对照：两值相等（或处于子模块中）→ 这是普通仓库检出。
>
> 用户是否已经在指令里表明过 worktree 偏好？如果没有，**先征得同意再创建**：
> 「需要我搭一个隔离 worktree 吗？它可以保护你当前分支不被改动。」
>
> 用户已经声明过偏好 → 直接照做，**不要再问一遍**（重复追问是噪音）。用户拒绝 → 就地工作，跳到 Step 2。

## Step 1: Create Isolated Workspace

**You have two mechanisms. Try them in this order.**
（你有两套机制，按这个顺序尝试。）

### 1a. Native Worktree Tools (preferred)

The user has asked for an isolated workspace (Step 0 consent). Do you already have a way to create a worktree? **If your runtime provides a native isolation mechanism, use it — its name varies by environment** (it might be a tool, a slash command, or a CLI flag). If you do, use it and skip to Step 2.

Native tools handle directory placement, branch creation, and cleanup automatically. Using `git worktree add` when you have a native tool creates phantom state your harness can't see or manage.

Only proceed to Step 1b if you have no native worktree tool available.

> 中文对照：
> 用户已同意隔离（Step 0 的同意）。这时先问自己：**我手上是否已经有创建隔离工作区的原生机制？**
>
> **若运行时有原生隔离工作区机制，则优先使用它——其名称随环境而异**（可能是一个工具、一条命令，或一个命令行开关）。有就走它，然后跳到 Step 2。
>
> **为什么原生优先**：原生机制自动处理目录位置、分支创建和清理，运行时全程知道这个工作区的存在。而你手上明明有原生工具却去调 `git worktree add`，会造成**运行时看不见、也管不了的幻影状态**（phantom state）——它不知道有这个目录、不会在完成时清理它、不会在切换上下文时把它算进去，后续所有基于"当前工作区"的判断都会失真。**这是最常见的错误。**
>
> 只有在**确实没有任何原生隔离机制**时，才进入 Step 1b。

### 1b. Git Worktree Fallback

**Only use this if Step 1a does not apply** — you have no native worktree tool available. Create a worktree manually using git.

> **只有在 Step 1a 不适用时**才用这条路——即你没有可用的原生隔离机制。此时用 git 手工创建 worktree。

#### Directory Selection

Follow this priority order. Explicit user preference always beats observed filesystem state.

1. **Check your instructions for a declared worktree directory preference.** If the user has already specified one, use it without asking.

2. **Check for an existing project-local worktree directory:**
   ```bash
   ls -d .worktrees 2>/dev/null     # Preferred (hidden)
   ls -d worktrees 2>/dev/null      # Alternative
   ```
   If found, use it. If both exist, `.worktrees` wins.

3. **If there is no other guidance available**, default to `.worktrees/` at the project root.

> 中文对照，**目录优先级**（用户的明确偏好永远压过文件系统观察到的状态）：
> 1. 先看指令里有没有声明过 worktree 目录偏好；有就直接用，不再问。
> 2. 再看项目里是否已存在 worktree 目录：`.worktrees`（首选，隐藏目录）与 `worktrees`（备选）。存在就用它；**两个都存在时，`.worktrees` 胜出**。
> 3. 都没有任何依据时，默认项目根的 `.worktrees/`。
>
> 顺序的逻辑：用户说过的话 > 项目里已存在的约定 > 默认值。**不要用"我觉得这个目录更好"去覆盖任何一层。**

#### Safety Verification (project-local directories only)

**MUST verify directory is ignored before creating worktree:**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:** Add to .gitignore, commit the change, then proceed.

**Why critical:** Prevents accidentally committing worktree contents to repository.

> 中文对照，**安全检查（仅针对项目内目录）**：
> **创建 worktree 之前，必须确认该目录已被 git 忽略。**
>
> 若**未被忽略**：先把它写进 `.gitignore`，**提交这次改动**，然后再继续。
>
> **为什么这一步是关键的**：worktree 目录里是**另一个完整的仓库工作副本**。如果它没被忽略，一次 `git add -A` 就会把你整个 worktree 的内容当成项目文件提交进仓库——数量级上是几千个文件、嵌套的仓库元数据，而且极难干净地回滚。这不是"注意一下"级别的风险，是必须用命令确认的硬门槛。
>
> 注意 `git check-ignore -q` 靠退出码判断，**不产生输出就是被忽略了**；命令没有回显不代表检测没跑。

#### Create the Worktree

```bash
# Determine path based on chosen location
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**Sandbox fallback:** If `git worktree add` fails with a permission error (sandbox denial), tell the user the sandbox blocked worktree creation and you're working in the current directory instead. Then run setup and baseline tests in place.

> 中文对照：
> 按选定的位置拼出路径，创建 worktree 并同时新建分支，然后切换进去。
>
> **沙箱降级**：如果 `git worktree add` 因权限错误（沙箱拒绝）失败，**如实告知用户**沙箱阻止了 worktree 创建、你改为在当前目录工作，然后在原地跑初始化与基线测试。**不要把失败吞掉假装成功了**——用户需要知道隔离没有生效。

## Step 2: Project Setup

Auto-detect and run appropriate setup:
（自动探测并执行对应的初始化。）

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

**读到这里的两种入口都适用**：Step 0 判定"已在 worktree 里"跳过来的，和 Step 1 刚建完过来的——都要在新工作区里把依赖装好，否则 Step 3 的基线测试必然失败，不是因为代码坏，而是因为环境空。

**没有任何清单文件时**（既无 `package.json` 也无 `Cargo.toml` 等）：跳过依赖安装，直接进入 Step 3。

## Step 3: Verify Clean Baseline

Run tests to ensure workspace starts clean:
（跑测试，确认工作区是从干净状态起步的。）

```bash
# Use project-appropriate command
npm test / cargo test / pytest / go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.
**If tests pass:** Report ready.

> 中文对照：**测试失败** → 报告失败内容，问用户是继续还是先排查。**测试通过** → 报告就绪。
>
> 为什么基线必须跑：如果这个工作区一开始就是红的，之后每一次失败都变成**歧义**——是你刚改坏的，还是本来就坏？基线跑一次的成本，换来后续全部失败信息的可解释性。**是否带着红基线继续干活，是用户的决定，不是你的。**

### Report

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

（报告格式：工作区就绪于 `<完整路径>`；测试通过（`<N>` 条，0 失败）；可以开始实现 `<功能名>`。）

## Quick Reference

| Situation | Action |
|-----------|--------|
| Already in linked worktree | Skip creation (Step 0) |
| In a submodule | Treat as normal repo (Step 0 guard) |
| Native worktree tool available | Use it (Step 1a) |
| No native tool | Git worktree fallback (Step 1b) |
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check instruction file, then default `.worktrees/` |
| Directory not ignored | Add to .gitignore + commit |
| Permission error on create | Sandbox fallback, work in place |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

> 中文对照（情境 → 动作）：
> - 已在关联 worktree 里 → 跳过创建（Step 0）。
> - 处于子模块中 → 当普通仓库处理（Step 0 的 submodule guard）。
> - 有原生隔离机制可用 → 用它（Step 1a）。
> - 没有原生机制 → 退回 git worktree（Step 1b）。
> - `.worktrees/` 已存在 → 用它（并确认已被忽略）。
> - `worktrees/` 已存在 → 用它（并确认已被忽略）。
> - 两者都存在 → 用 `.worktrees/`。
> - 两者都不存在 → 先查指令文件，没有则默认 `.worktrees/`。
> - 目录未被忽略 → 加进 .gitignore 并提交。
> - 创建时报权限错误 → 沙箱降级，就地工作。
> - 基线测试失败 → 报告失败并询问用户。
> - 没有 package.json / Cargo.toml → 跳过依赖安装。

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'm obviously not in a worktree — no need to check" | Run Step 0. Harness-created isolation and submodules both fool eyeballing; the detection commands settle it. |
| "`git worktree add` is quicker than hunting for a native tool" | A native tool owns placement, branching, and cleanup. Bypassing it is the #1 mistake — it creates phantom state your harness can't see or manage. |
| "The worktree directory is surely ignored already" | Run `git check-ignore`. An unignored worktree directory commits the whole tree into the repo. |
| "Any directory name works" | Explicit instructions beat an existing project-local directory, which beats the `.worktrees/` default. |
| "The workspace is fresh — baseline tests can wait" | A dirty baseline makes every later failure ambiguous. Run the tests now; proceeding past failures is your human partner's call. |

> 中文对照：
> - **"我很显然不在 worktree 里，不用查"** → 跑 Step 0。运行时创建的隔离和子模块这两种情况，肉眼都判断不出来；让检测命令来定论。
> - **"`git worktree add` 比去找原生工具快"** → 原生工具负责目录位置、分支创建和清理。绕过它就是头号错误——它造出运行时看不见、也管不了的幻影状态。
> - **"worktree 目录肯定已经被忽略了"** → 跑 `git check-ignore`。没被忽略的 worktree 目录会把整棵树提交进仓库。
> - **"目录叫什么都行"** → 明确指令 > 项目里已存在的目录 > `.worktrees/` 默认值。
> - **"工作区是新的，基线测试可以等等"** → 脏基线会让之后每一次失败都变成歧义。现在就跑；带着失败继续往下走，是用户的决定。

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 隔离工作区就绪，开始写实现 | `test-driven-development`——隔离只是场地，纪律在那边 |
| 实现过程中出现 bug / 测试失败 | `systematic-debugging`——先找根因 |
| 要执行一份实现计划、任务较多 | 本 skill 负责场地，执行循环回到计划与实现类 skill |
| 实现完成，准备合入 | `finishing-a-development-branch`——它依赖本 skill 建立的隔离工作区与 `.worktrees/` 归属判断 |
| 准备声称"已完成" | `verification-before-completion`——**强制** |

**本 skill 管"在哪干活"**——它不改任何业务代码，只负责让工作在正确的、运行时看得见的隔离场地里发生。**没有经过本 skill 就直接在主分支上开工，等于把用户当前的工作当成了实验场地。**
