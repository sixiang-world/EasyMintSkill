---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---

# Writing Skills

## Overview

**写 skill 就是 TDD 用在"流程文档"上。**

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.
（没看过 Agent 在没有这个 skill 时怎么失败，你就不知道这个 skill 教的东西对不对。）

**REQUIRED BACKGROUND:** 你必须先理解 `test-driven-development`。那个 skill 定义了 RED-GREEN-REFACTOR 循环；本 skill 把它适配到文档。

**Personal skills 放在运行时约定的 skills 目录**——项目级放 `.agentskill/skills/`（见 `methodology-core` 的资源位置约定）。**用户级目录各运行时约定不同，按所用运行时自己的约定放置即可，不要照搬别家的路径。**

**Official guidance:** 官方 skill 编写最佳实践见 [references/anthropic-best-practices.md](references/anthropic-best-practices.md)，与本文的 TDD 取向互补。

## What is a Skill?

A **skill** is a reference guide for proven techniques, patterns, or tools.

**Skills are:** Reusable techniques, patterns, tools, reference guides

**Skills are NOT:** Narratives about how you solved a problem once

## When to Create a Skill

**Create when:** 技法当初对你并不显然 / 跨项目会反复查 / 适用面广（非项目专属）/ 别人也用得上。

**Don't create for:** 一次性方案 / 别处已有权威文档的通用做法 / 项目专属约定（放指令文件）/ 机械性约束（**能用校验强制执行的，就自动化；文档留给需要判断的地方**）。

## TDD Mapping for Skills

写 skill 与写代码一一对应。**下表保留英文原文**，右侧附中文口径。

| TDD Concept | Skill Creation | 中文口径 |
|-------------|----------------|---------|
| **Test case** | Pressure scenario with subagent | 测试用例 = 带压力的场景，跑在子 Agent 上 |
| **Production code** | Skill document (SKILL.md) | 产出物 = skill 文档本身 |
| **Test fails (RED)** | Agent violates rule without skill (baseline) | 没有 skill 时 Agent 违规 = 失败基线 |
| **Test passes (GREEN)** | Agent complies with skill present | 有 skill 时 Agent 遵守 = 通过 |
| **Refactor** | Close loopholes while maintaining compliance | 堵漏洞，同时保住已通过的合规 |
| **Write test first** | Run baseline scenario BEFORE writing skill | **先跑基线场景，再写 skill** |
| **Watch it fail** | Document exact rationalizations agent uses | 逐字记录 Agent 用的合理化借口 |
| **Minimal code** | Write skill addressing those specific violations | 只针对观测到的违规写，不写假想情况 |
| **Watch it pass** | Verify agent now complies | 验证 Agent 现在遵守 |
| **Refactor cycle** | Find new rationalizations → plug → re-verify | 找新借口 → 堵住 → 再验证 |

The entire skill creation process follows RED-GREEN-REFACTOR.

## Skill Types

**Technique**——有步骤可循的具体方法；**Pattern**——看问题的思维方式；**Reference**——API 文档、语法指南、工具说明。

## Directory Structure

```
skills/
  skill-name/
    SKILL.md              # Main reference (required)
    references/           # 下沉的重型参考（可选）
    supporting-file.*     # Only if needed
```

**Flat namespace** - all skills in one searchable namespace

**Separate files for（下沉到 references/）:** **Heavy reference** (100+ lines；API docs、完整语法) · **Reusable tools** (scripts、utilities、templates)

**Keep inline（留在 SKILL.md 内）:** 原则与概念 · 代码模式（< 50 行） · 其他一切

**引用只一层深**：所有参考文件都从 SKILL.md **直接**链接。嵌套引用（SKILL.md → a.md → b.md）会让 Agent 用 `head -100` 预览而不读完整个文件，拿到不完整信息。本 skill 的三份 references 就是直接从 SKILL.md 链出的一层。

## 文件组织：三种布局与选用判据

上面给的是通用规则，落到具体 skill 时有三种典型形态。**先判断属于哪一种，再决定建哪些文件**——形态选错，要么该下沉的没下沉（SKILL.md 臃肿），要么不该拆的拆了（引用一层深被破坏）。

### 自包含型（Self-Contained）

```
defense-in-depth/
  SKILL.md    # 全部内容 inline
```

**何时用**：全部内容能塞进去，不需要重型参考。

### 带可复用工具型（Skill with Reusable Tool）

```
condition-based-waiting/
  SKILL.md    # 概览 + 模式
  example.ts  # 可适配的、能跑的工具代码
```

**何时用**：那个工具是**可复用的代码**，而不只是叙述性示例。用户会想直接抄走去改。

### 带重型参考型（Skill with Heavy Reference）

```
pptx/
  SKILL.md       # 概览 + 工作流
  pptxgenjs.md   # 600 行 API 参考
  ooxml.md       # 500 行 XML 结构
  scripts/       # 可执行工具
```

**何时用**：参考材料**大到不适合 inline**（几百行的 API 文档、完整语法表、数据结构说明）。

> **判据一句话**：内容长度决定是否下沉（>100 行就该下沉），内容**性质**决定沉到哪——可执行代码放 `scripts/`，被查阅的文档放 `references/`，要被引用/改写的示例放 `assets/`。

## SKILL.md Structure

**Frontmatter (YAML):**
- Two required fields: `name` and `description`
- **Max 1024 characters total**
- `name`: Use letters, numbers, and hyphens only (no parentheses, special chars)
- `description`: Third-person, describes **ONLY when to use** (NOT what it does)
  - Start with `Use when...` to focus on triggering conditions
  - Include specific symptoms, situations, and contexts
  - **NEVER summarize the skill's process or workflow**（见 SDO 一节）
  - Keep under 500 characters if possible

**正文推荐结构**（不是硬模板，缺项要想清楚为什么缺）：

| 章节 | 放什么 |
|---|---|
| `## Overview` | 这是什么 + Core principle，1-2 句 |
| `## When to Use` | 症状与场景清单 + **When NOT to use**；决策非显然时才配一张小流程图 |
| `## Core Pattern` | 技法/模式类用 before/after 代码对比 |
| `## Quick Reference` | 表格或清单，供扫读 |
| `## Implementation` | 简单模式 inline；重型参考或可复用工具**链到文件** |
| `## Common Mistakes` | 会出什么错 + 怎么修 |
| `## 与其他 skill 的关系` | 表格：什么情境转入哪个 skill |
| `## Real-World Impact` | 具体结果（可选） |

完整模板（含可直接抄的骨架）见 [references/testing-skills-with-subagents.md](references/testing-skills-with-subagents.md) 的 SKILL.md Structure 一节。

## Skill Discovery Optimization (SDO)

**Critical for discovery:** Future agents need to FIND your skill

### 1. Rich Description Field

**Purpose:** Agent 读 description 来决定该不该载入这个 skill。让它回答："我现在该读这个 skill 吗？"

**Format:** Start with "Use when..." to focus on triggering conditions

**CRITICAL: Description = When to Use, NOT What the Skill Does**

description **只写触发条件**，绝不概述 skill 的流程或工作流。

**Why this matters:** 实测发现——当 description 概述了 workflow，Agent 会**照 description 走捷径，而不去读正文**。一个写着 "code review between tasks" 的 description 导致 Agent 只做**一次** review，而 skill 正文的流程图明确要求**两次**（先 spec 合规，再代码质量）。当 description 改成只写 "Use when executing implementation plans with independent tasks"（不带任何 workflow 概述），Agent 就正确地读了流程图并执行了两阶段 review。

**The trap:** 概述 workflow 的 description 会变成 Agent 会走的短路。正文于是成了 Agent 会跳过的文档。

**八行 description 反例 / 正例对照**（**原文照录，逐行对照读**）：

```yaml
# ❌ BAD: Summarizes workflow - agents may follow this instead of reading skill
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ❌ BAD: Too much process detail
description: Use for TDD - write test first, watch it fail, write minimal code, refactor

# ✅ GOOD: Just triggering conditions, no workflow summary
description: Use when executing implementation plans with independent tasks in the current session

# ✅ GOOD: Triggering conditions only
description: Use when implementing any feature or bugfix, before writing implementation code

# ❌ BAD: Too abstract, vague, doesn't include when to use
description: For async testing

# ❌ BAD: First person
description: I can help you with async tests when they're flaky

# ❌ BAD: Mentions technology but skill isn't specific to it
description: Use when tests use setTimeout/sleep and are flaky

# ✅ GOOD: Starts with "Use when", describes problem, no workflow
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently

# ✅ GOOD: Technology-specific skill with explicit trigger
description: Use when using React Router and handling authentication redirects
```

**Content:**
- Use concrete triggers, symptoms, and situations that signal this skill applies
- Describe the *problem* (race conditions, inconsistent behavior) not *language-specific symptoms* (setTimeout, sleep)
- Keep triggers technology-agnostic unless the skill itself is technology-specific
- If skill is technology-specific, make that explicit in the trigger
- Write in third person (injected into system prompt)
- **NEVER summarize the skill's process or workflow**

### 2. Keyword Coverage

用 Agent 会去搜的词：
- Error messages: "Hook timed out", "ENOTEMPTY", "race condition"
- Symptoms: "flaky", "hanging", "zombie", "pollution"
- Synonyms: "timeout/hang/freeze", "cleanup/teardown/afterEach"
- Tools: Actual commands, library names, file types

### 3. Descriptive Naming

**Use active voice, verb-first**（主动语态，动词优先）：
- ✅ `creating-skills` not `skill-creation`
- ✅ `condition-based-waiting` not `async-test-helpers`
- ✅ `using-skills` not `skill-usage`
- ✅ `flatten-with-flags` > `data-structure-refactoring`
- ✅ `root-cause-tracing` > `debugging-techniques`

**Gerunds (-ing) work well for processes:** `creating-skills`, `testing-skills`, `debugging-with-logs`——主动，描述你正在做的动作。

### 4. Token Efficiency (Critical)

**Problem:** getting-started 与高频 skill 会载入**每一次**对话。每个 token 都要算账。

**Target word counts（长度预算）:**
- getting-started workflows: **<150 words each**
- Frequently-loaded skills: **<200 words total**
- Other skills: **<500 words** (still be concise)
- **SKILL.md body 总量 <500 行**（超了就往 references 下沉）

**四种压缩手法**（省的是反复载入的成本）：

| 手法 | ❌ 反例 | ✅ 正例 |
|---|---|---|
| **细节交给 --help** | 在 SKILL.md 里罗列全部 flag | "支持多种模式与过滤，`--help` 看细节" |
| **交叉引用而非复述** | 把别的 skill 的 20 行 workflow 抄一遍 | 一句话 + `REQUIRED: Use [other-skill]` |
| **压缩示例** | 42 词的长对话示例 | 20 词的最小示例 |
| **删冗余** | 重复别处内容 / 解释命令已自明的行为 / 同一模式给多个例子 | 一个优秀例子 |

**Verification:**
```bash
wc -w skills/path/SKILL.md   # getting-started: <150 each; 高频: <200 total
wc -l skills/path/SKILL.md   # SKILL.md body: <500 lines
```

### 5. Cross-Referencing Other Skills

**When writing documentation that references other skills:** 只用 skill 名 + 显式要求标记。

- ✅ Good: `**REQUIRED SUB-SKILL:** Use test-driven-development`
- ✅ Good: `**REQUIRED BACKGROUND:** You MUST understand systematic-debugging`
- ❌ Bad: `See skills/testing/test-driven-development` (unclear if required)
- ❌ Bad: `@skills/testing/test-driven-development/SKILL.md` (force-loads, burns context)

**Why no @ links:** `@` 语法会**立即强制载入**文件，在你还需要之前就烧掉大量上下文。

## The Iron Law (Same as TDD)

```
NO SKILL WITHOUT A FAILING TEST FIRST
```

**This applies to NEW skills AND EDITS to existing skills.**

Write skill before testing? Delete it. Start over.
Edit skill without testing? Same violation.

**No exceptions:** Not for "simple additions" · Not for "just adding a section" · Not for "documentation updates" · Don't keep untested changes as "reference" · Don't "adapt" while running tests · **Delete means delete**

**REQUIRED BACKGROUND:** `test-driven-development` 解释了这条为什么成立。同一套原则适用于文档。

## Testing All Skill Types

不同 skill 类型要不同的测法：

| 类型 | 测什么 | 通过标准 |
|---|---|---|
| **Discipline-Enforcing**（规则类） | 学术问题 + **压力场景 + 多压力叠加** | Agent 在**最大压力**下仍遵守规则 |
| **Technique**（how-to 类） | 应用场景 + 变体场景 + 信息缺口测试 | Agent 能把技法用在新场景上 |
| **Pattern**（心智模型类） | 识别场景 + 应用场景 + **反例** | Agent 能判断何时/**不**该用 |
| **Reference**（文档/API 类） | 检索场景 + 应用场景 + 缺口测试 | Agent 找得到并正确套用 |

**各类的具体测试步骤与完整判据**见 [references/testing-skills-with-subagents.md](references/testing-skills-with-subagents.md)。

## Common Rationalizations for Skipping Testing

八条"不测也行"的借口，逐条驳回（**保留英文原文**）：

| Excuse | Reality |
|--------|---------|
| "Skill is obviously clear" | Clear to you ≠ clear to other agents. Test it. |
| "It's just a reference" | References can have gaps, unclear sections. Test retrieval. |
| "Testing is overkill" | Untested skills have issues. Always. 15 min testing saves hours. |
| "I'll test if problems emerge" | Problems = agents can't use skill. Test BEFORE deploying. |
| "Too tedious to test" | Testing is less tedious than debugging bad skill in production. |
| "I'm confident it's good" | Overconfidence guarantees issues. Test anyway. |
| "Academic review is enough" | Reading ≠ using. Test application scenarios. |
| "No time to test" | Deploying untested skill wastes more time fixing it later. |

**All of these mean: Test before deploying. No exceptions.**

## Match the Form to the Failure

**动笔之前，先给基线失败分类。** 同一副"形式"能给一类失败上膛，却会在另一类上**实测倒退**。

| Baseline failure | Right form | Wrong form |
|---|---|---|
| Skips/violates a rule under pressure (knows better, does it anyway)<br>**知道规则却在压力下违反** | Prohibition + rationalization table + red flags (见 Bulletproofing)<br>**禁令 + 合理化表 + Red Flags** | Soft guidance ("prefer...", "consider...")<br>**软性引导（"建议…""可考虑…"）** |
| Complies, but output has the wrong shape (bloated prompt, buried verdict, restated spec)<br>**遵守了，但产出形状错**（prompt 臃肿、结论被埋、复述了 spec） | Positive recipe or contract: state what the output IS — its parts, in order<br>**正向配方/契约：直接说明产出「是」什么——由哪些部分构成、按什么顺序** | Prohibition list ("don't restate", "never narrate")<br>**禁令清单（"不要复述""永不叙述"）** |
| Omits a required element from something they already produce<br>**在已经会做的产出里漏了必备要素** | Structural: REQUIRED field or slot in the template they fill in<br>**结构化：在模板里加 REQUIRED 字段或槽位** | Prose reminders near the template<br>**模板旁边的散文提醒** |
| Behavior should depend on a condition<br>**行为应当依条件而变** | Conditional keyed to an observable predicate ("if the brief exists, reference it")<br>**挂在可观测谓词上的条件句** | Unconditional rule + exemption clauses<br>**无条件规则 + 豁免条款** |

**Why prohibitions backfire on shaping problems:** 在"让 prompt 自包含"这类竞争性激励下，Agent 会跟 "don't X" 讨价还价。在 dispatch-prompt 指导语的**正面交锋措辞测试**中，禁令组产出的多余内容**明显多于**配方组（两组分布完全分离），甚至**比无指导的对照组还差**。所以：**微测你自己的场景，别假设；但永远不要默认去抓禁令。** 配方没有可讨价的空间——产出要么符合声明的形状，要么不符合。

**每条规律背后的实测数据与完整分类判据**（几类指令、各类得分与样本量、改写前的判断顺序）见 [references/instruction-phrasing.md](references/instruction-phrasing.md)。

**Rules for whichever form you pick:**
- **No nuance clauses.** "Don't X unless it matters" 会把谈判重新打开——给一个**已经赢了的配方**追加一条 nuance 从句，就把结果从"稳定"降到"嘈杂"。真有例外，就把它写成挂在可观测谓词上的独立条件句。
- **Exemption clauses don't scope.** "This limit doesn't apply to code blocks" 依然会压制代码块。如果产出中某部分必须豁免，**重构结构让规则够不到它**，而不是加豁免条款。

## Bulletproofing Skills Against Rationalization

反合理化（防钻空子）四件套。**适用范围**：本套工具针对**纪律失败**——Agent 知道规则却在压力下跳过。对"形状错误"或"漏要素"，禁令式加固会倒退，请改用上一节的对应形式。

1. **Close Every Loophole Explicitly**——不要只说规则，要把具体绕法逐条禁掉：

```markdown
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```

（坏的写法是只留第一句 `Write code before test? Delete it.`——它给"当参考留着"留了口子。）

2. **Address "Spirit vs Letter" Arguments**——把基础原则前置：

```markdown
**Violating the letter of the rules is violating the spirit of the rules.**
```

这一句切断了一整类"I'm following the spirit"的合理化。

3. **Build Rationalization Table**——把基线测试里抓到的每一个借口写进表（**实测得来，不是想象**）：

```markdown
| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Tests after achieve same goals" | Tests-after = "what does this do?" Tests-first = "what should this do?" |
```

4. **Create Red Flags List**——让 Agent 在合理化时能自检：

```markdown
## Red Flags - STOP and Start Over

- Code before test
- "I already manually tested it"
- "Tests after achieve the same purpose"
- "It's about spirit not ritual"
- "This is different because..."

**All of these mean: Delete code. Start over with TDD.**
```

另外：把**违规前兆**加进 description：

```yaml
description: use when implementing any feature or bugfix, before writing implementation code
```

### Pressure Types（压力类型）

纪律类 skill 要能扛住压力。压力分七类，**最好的测试组合 3+ 种**：

| Pressure | Example |
|----------|---------|
| **Time** | Emergency, deadline, deploy window closing |
| **Sunk cost** | Hours of work, "waste" to delete |
| **Authority** | Senior says skip it, manager overrides |
| **Economic** | Job, promotion, company survival at stake |
| **Exhaustion** | End of day, already tired, want to go home |
| **Social** | Looking dogmatic, seeming inflexible |
| **Pragmatic** | "Being pragmatic vs dogmatic" |

**Psychology note:** 理解说服技巧**为什么**有效，能帮你系统性地应用它们。研究基础（Cialdini, 2021; Meincke et al., 2025——权威、承诺、稀缺、社会认同、统一性）见 [references/persuasion-principles.md](references/persuasion-principles.md)。

## RED-GREEN-REFACTOR for Skills

| 阶段 | 做什么 | 产出 |
|---|---|---|
| **RED** | **不带 skill** 跑压力场景。逐字记录：它们选了什么？用了什么借口（**逐字**）？哪种压力触发了违规？ | 失败基线与借口清单 |
| **GREEN** | **只针对**上面那些具体借口写最小 skill，**不为假想情况加内容**。同一批场景**带上 skill** 再跑。 | 能通过基线的 skill |
| **REFACTOR** | Agent 又找到新借口 → 加显式反制 → 重测，直到打不穿。 | Bulletproof 的 skill |

RED 就是"watch the test fail"——你必须**先看见** Agent 自然会怎么做，再写 skill。

### Micro-Test Wording Before Full Scenarios

完整压力场景是最后一道门，但**每次迭代都慢且贵**。先用 micro-test 验措辞本身：

1. **One fresh-context sample per call** — 一次裸 API 调用，或（没有 API 访问时）一次性的单发子 Agent。System prompt = 该指导语真实存在的语境（**整个 skill 或 prompt 模板，而不是孤立的这句指导语**）；user message = 一个会诱发该失败的任务。
2. **Always include a no-guidance control.** 如果对照组**没有**表现出这个失败——那就没什么可修的，**停手，别写这条指导语**。
3. **5+ reps per variant.** 单个样本会说谎。
4. **Manually read every flagged match.** 你可以用程序打分，但要**人工读每一条命中**——模板回显和被引用的反例都会伪装成命中，纯自动计数会同时高估失败与成功。
5. **Variance is a metric.** 指导语生效时，多次重复会收敛到同一形状。**五次重复出现五种不同解读，说明措辞没有约束力**——先收形式，再考虑加字数。

**Micro-test 验的是措辞；对纪律类 skill，它不替代压力场景。**

**Testing methodology:** 完整测试方法（怎么写压力场景、系统性地堵洞、元测试技巧、完整 Checklist）见 [references/testing-skills-with-subagents.md](references/testing-skills-with-subagents.md)。

**现成的攻击语料：** 如果被测的是**强制触发层**这类"必须抢在动作之前生效"的技能，上面七类压力还缺一种最关键的——**用户主动诱导你跳过流程**。9 个攻击 prompt（裸命名、礼貌点名、时间压力、伪造"流程已完成"、伪官方推荐……）＋ "触发 + 顺序"两层判定思路，见 [references/trigger-test-prompts.md](references/trigger-test-prompts.md)。核心判据：**技能调用之前出现任何动手类工具调用，即失败**——提前动手 = 流程已被绕过。

## Flowchart Usage

```
需要展示信息吗？
├─ 否 → 不需要任何图
└─ 是 → 这是一个"我可能会判断错"的决策点吗？
        ├─ 是 → 用小而内联的流程图
        └─ 否 → 用 markdown（表格 / 列表 / 代码块）
```

**Use flowcharts ONLY for:** Non-obvious decision points · Process loops where you might stop too early · "When to use A vs B" decisions

**Never use flowcharts for:** Reference material → Tables, lists · Code examples → Markdown blocks · Linear instructions → Numbered lists · Labels without semantic meaning (step1, helper2)

**渲染支持**：流程图用 graphviz dot 语法写。样式规则见 [graphviz-conventions.dot](graphviz-conventions.dot)。

**渲染成 SVG 给用户看**：用本目录的 `render-graphs.js` 把一个技能里的流程图渲染成 SVG：

```bash
./render-graphs.js ../some-skill            # 每张图单独输出
./render-graphs.js ../some-skill --combine  # 所有图合并成一张 SVG
```

它扫描 SKILL.md 里的 dot 代码块，调 `dot` 转 SVG，输出到 `diagrams/`。脚本在缺失 graphviz 时会给出安装提示并优雅失败（已为 Windows 适配，不依赖 `which`）。

**先判断这张图是否真的必要**——非显然的决策点才值得一张图。仅仅为"好看"加的图，直接删掉换成表格更省 token。图本身要看，但**不要为了看图而画图**。

## Code Examples

**One excellent example beats many mediocre ones**

**Good example:** Complete and runnable · Well-commented explaining WHY · From real scenario · Shows pattern clearly · Ready to adapt (not generic template)

**Don't:** Implement in 5+ languages · Create fill-in-the-blank templates · Write contrived examples

选最相关的语言即可（测试技法 → TypeScript/JavaScript；系统调试 → Shell/Python；数据处理 → Python）。**你很擅长移植——一个优秀的例子就够了。**

## Anti-Patterns

| ❌ 反模式 | 例子 | 为什么坏 |
|---|---|---|
| **Narrative Example** | "In session 2025-10-03, we found empty projectDir caused..." | 太具体，不可复用（skill 不是日记） |
| **Multi-Language Dilution** | example-js.js, example-py.py, example-go.go | 每个语言版本都平庸，还多一份维护负担 |
| **Code in Flowcharts** | `step1 [label="import fs"]; step2 [label="read file"];` | 不能复制粘贴，难读 |
| **Generic Labels** | helper1, helper2, step3, pattern4 | 标签应当有语义 |

## STOP: Before Moving to Next Skill

**After writing ANY skill, you MUST STOP and complete the deployment process.**

**Do NOT:**
- Create multiple skills in batch without testing each
- Move to next skill before current one is verified
- Skip testing because "batching is more efficient"

**The deployment checklist below is MANDATORY for EACH skill.**

Deploying untested skills = deploying untested code.

## Skill Creation Checklist (TDD Adapted)

**IMPORTANT: Create a todo for EACH checklist item below.**（下面每一项都建一个 todo，**清单不做 todo 追踪 = 步骤必被跳过，每次都一样**。）

**RED Phase - Write Failing Test:**
- [ ] Create pressure scenarios (3+ combined pressures for discipline skills)
- [ ] Run scenarios WITHOUT skill - document baseline behavior verbatim
- [ ] Identify patterns in rationalizations/failures

**GREEN Phase - Write Minimal Skill:**
- [ ] Name uses only letters, numbers, hyphens (no parentheses/special chars)
- [ ] YAML frontmatter with required `name` and `description` fields (max 1024 chars)
- [ ] Description starts with "Use when..." and includes specific triggers/symptoms
- [ ] Description written in third person
- [ ] Description does NOT summarize the skill's process or workflow
- [ ] Keywords throughout for search (errors, symptoms, tools)
- [ ] Clear overview with core principle
- [ ] Address specific baseline failures identified in RED
- [ ] Guidance form matches the failure type (see Match the Form to the Failure)
- [ ] For behavior-shaping guidance: wording micro-tested against a no-guidance control (5+ reps, every flagged match read manually) — N/A for pure reference skills
- [ ] Code inline OR link to separate file
- [ ] One excellent example (not multi-language)
- [ ] Run scenarios WITH skill - verify agents now comply

**REFACTOR Phase - Close Loopholes:**
- [ ] Identify NEW rationalizations from testing
- [ ] Add explicit counters (if discipline skill)
- [ ] Build rationalization table from all test iterations
- [ ] Create red flags list
- [ ] Re-test until bulletproof

**Quality Checks:**
- [ ] Small flowchart only if decision non-obvious
- [ ] Quick reference table
- [ ] Common mistakes section
- [ ] No narrative storytelling
- [ ] Supporting files only for tools or heavy reference
- [ ] References are one level deep from SKILL.md
- [ ] SKILL.md body under 500 lines

**Deployment:**
- [ ] Commit skill

## Discovery Workflow

How future agents find your skill:

1. **Encounters problem** ("tests are flaky")
2. **Searches skills** (greps descriptions, browses categories)
3. **Finds SKILL** (description matches)
4. **Scans overview** (is this relevant?)
5. **Reads patterns** (quick reference table)
6. **Loads example** (only when implementing)

**Optimize for this flow** - put searchable terms early and often.

## 与其他 skill 的关系

| 情境 | 转入 |
|---|---|
| 要写的是纪律类 skill，需要理解 RED-GREEN-REFACTOR 的本体 | `test-driven-development`——**REQUIRED BACKGROUND**，本 skill 是它的文档版 |
| 需要一个真实的完整测试战役作参照 | [references/testing-skills-with-subagents.md](references/testing-skills-with-subagents.md) |
| 要设计有说服力的措辞（权威/承诺/稀缺等） | [references/persuasion-principles.md](references/persuasion-principles.md) |
| 要对齐官方 skill 编写规范（命名/渐进披露/评估） | [references/anthropic-best-practices.md](references/anthropic-best-practices.md) |
| 新建的是要把多 Agent 串起来的编排 skill | `subagent-driven-development`——学习它的两阶段审查与 ledger 设计取舍 |
| skill 要派发子 Agent | `dispatching-parallel-agents`——并发派发的写法约束 |
| 新 skill 里含"完成声明"类要求 | `verification-before-completion`——铁律与反合理化表的范例 |
| 新 skill 里含"收到反馈"类要求 | `receiving-code-review`——Forbidden Responses 与 push back 的范例 |
| 新 skill 显得过度工程、抽象过多 | `ponytail`——先过 YAGNI 那关 |

**本 skill 管"skill 本身怎么写、怎么测";它不管"skill 里写什么方法论"——那是各领域 skill 的事。**
