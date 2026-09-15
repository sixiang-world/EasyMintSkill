# 第三方资产来源与授权声明

本目录（`assets/`）内含两部分第三方来源资产：**品牌设计规范库**与**种子 HTML 模板**。使用前请阅读本声明。

---

## 1. 品牌设计规范库（`assets/brand-tokens/`）

### 内容与用途

收录 74 个知名品牌的设计系统分析（`DESIGN.md`）。用途是**设计风格参考**——在用户提出「做个像 X 那样的界面」时，提供可解析的配色、排版、圆角、间距 token 作为起点。

### 来源

这些分析文档为**第三方独立撰写**的设计系统解读，由 EasyMint 项目（https://github.com/tianemon/EasyMint）整理分发，本包据此收录。

### 重要限制

- **非官方资产**。所有文档均**不是**对应品牌的官方设计规范，也未经其授权或背书。文中 `name` 字段使用 `<品牌>-Inspired` 形式，仅为风格归类的标识。
- **商标归各自所有者**。文档中出现的品牌名称、商标、logo、产品截图等，权利归其各自所有者。
- **字体不随包分发**。文档中提及的商业字体（如 Klim Type Foundry 的 Sohne、Apple 的 SF Pro 等）**均未包含在本包内**；文档仅描述其视觉特征并给出开源替代建议（如 Inter）。商用前须自行取得字体授权。

### 使用约束

使用者须自行遵守：

- 只提取**设计语言**（色值、字号梯度、间距节奏、圆角体系）作为风格参照；
- **不得**复制品牌名称、商标、logo 用于自己的产品；
- **不得**暗示与对应品牌存在关联、赞助或背书；
- 若需商用，请自行评估商标与不正当竞争风险，必要时咨询法律意见。

> 本包仅作方法论组件分发这些参考文档；因使用这些文档而产生的**任何商标、版权或不正当竞争纠纷，由使用者自行承担**。

---

## 2. 种子 HTML 模板（`assets/templates/`）

### 内容

4 个单文件 HTML 起步模板：`landing.html`、`dashboard.html`、`form.html`、`detail.html`。

### 来源与授权

由 EasyMint 项目制作并整理，纯 CSS（无外部依赖、无框架、无第三方库），随本包以与仓库相同的许可分发。

模板中的设计变量体系（`--bg` / `--surface` / `--fg` / `--accent` 等）为通用 CSS 自定义属性写法，不含专有内容。

### 使用方式

直接作为新页面的起点：替换 `:root` 变量以套用视觉风格，替换正文为真实产品文案。模板中的示例文案仅为占位，交付前须全部替换。

---

## 2.5 方法论来源：obra/superpowers

本包的**执行期纪律 skill 与部分机制**，在设计与措辞上借鉴（部分段落直接采用）了开源项目 **Superpowers**：

- 项目：https://github.com/obra/superpowers
- 作者：Jesse Vincent 与 Prime Radiant（https://primeradiant.com）
- 许可：**MIT License**

### 借鉴范围

下列 skill 与机制源自或改编自 Superpowers：

| 本包内容 | Superpowers 对应 |
|---|---|
| `using-easymint`（强制触发层 + Red Flags 表） | `using-superpowers` |
| `kickoff`（三路径分类 + HARD-GATE 批准闸门 + 单向升级 + 规格自审 + Visual Companion 及其服务端脚本） | `brainstorming` 及附属 `visual-companion.md` / `scripts/` |
| `test-driven-development` | 同名 skill |
| `systematic-debugging`（含 root-cause-tracing / defense-in-depth / condition-based-waiting） | 同名 skill 及同名 references |
| `verification-before-completion`（Iron Law + Gate Function） | 同名 skill |
| `using-git-worktrees` | 同名 skill |
| `finishing-a-development-branch` | 同名 skill |
| `requesting-code-review`（含 `references/code-reviewer.md`） | 同名 skill |
| `receiving-code-review`（Response Pattern + 反服从机制） | 同名 skill |
| `dispatching-parallel-agents` | 同名 skill |
| `writing-plans` / `executing-plans` | 同名 skill |
| `subagent-driven-development`（两阶段 review / ledger / breaker / 三个 prompt 模板 / 三个脚本） | 同名 skill 及附属文件 |
| `writing-skills`（含五份 references 与 graphviz 约定） | 同名 skill 及同名 references |
| `writing-skills/references/instruction-phrasing.md` | `docs/superpowers/specs/2026-06-10-positive-instruction-redesign-design.md`（实测数据与五类指令分类） |
| `writing-skills/references/trigger-test-prompts.md` | `tests/explicit-skill-requests/`（9 个触发攻击 prompt + "触发 + 顺序"两层判定） |
| `writing-skills/references/testing-campaign-example.md` | `CLAUDE_MD_TESTING.md`（指令措辞 A/B 测试战役实例） |
| `writing-skills/render-graphs.js` | 同名脚本（graphviz 渲染） |
| `easymint-core` 的 Evaluator 两阶段升级、fix loop、Rulings not stalls | `subagent-driven-development` + `writing-skills` |
| 各 skill 的 `Red Flags` 与 `Common Rationalizations` 表 | 上述各 skill 的同名段落 |

### 改编说明

本包的改编包括：正文改写为中文（`Red Flags` / `Common Rationalizations` 表与 prompt 模板保留英文原文，因这些措辞的心理阻断效果依赖原文）；去除平台专属内容（原项目的多 harness 适配层、特定运行时的工具名与配置路径）；项目状态目录统一为本包约定的 `.agentskill/`。

三份源自 Superpowers 的测试与实测参考（`instruction-phrasing.md` / `trigger-test-prompts.md` / `testing-campaign-example.md`）另做了以下改编：

- **`instruction-phrasing.md`**：忠实转述实测数据与五类指令分类，未改动任何数值；仅移除其中的具体产品名与模型名（如原文"Codex re-reads SKILL.md ~500× per session"里的产品名改为"长会话中"）。
- **`trigger-test-prompts.md`**：**只吸收攻击角度与判定思路，不迁移原版的平台专属 runner 调用方式**。9 段攻击语料逐字保留（翻译会削弱其作为测试素材的诱导力）；其中的原版产品名已在正文与投喂 payload 中替换为中立表述。
- **`testing-campaign-example.md`**：被测对象所在的常驻指令文件名与技能库目录路径统一抽象为 `<INSTRUCTIONS_FILE>` / `<SKILLS_DIR>` 占位符（因不同运行时的叫法与路径各不相同）；四个压力场景、五个措辞变体、四步协议、镜像判据与结论全部保留，未做删减。

`kickoff` 的 Visual Companion 服务端脚本另做了以下改编：

- **移除第三方品牌注入**：原脚本会向页面注入作者品牌 logo，且该 logo 由第三方服务器（`primeradiant.com`）提供。本包**完全移除了品牌注入与外链请求**，服务端不再发起任何外部网络访问。
- **移除遥测相关开关**：原脚本读取遥测禁用环境变量以决定是否显示品牌，本包连同该逻辑一并移除（无遥测即无需开关）。
- **环境变量与路径中立化**：`BRAINSTORM_*` → `KICKOFF_*`，`.superpowers/brainstorm/` → `.agentskill/kickoff/`，服务端标识改为 `kickoff`。
- **平台适配改为通用规则**：原文档按具体产品名列出 4 种启动方式，本包改写为"判断你的运行环境是否会回收后台进程"的通用决策规则，保留技术判据而去掉产品点名。
- **修复 Windows 路径处理**：新增 `cygpath` 路径规范化，使 Git Bash 风格的 `/c/Users/...` 能正确转为 Node 可解析的原生路径（否则会生成 `\c\Users\...` 畸形目录）。

**Superpowers 的 MIT 许可要求保留其版权声明**。若你分发本包，请一并保留本节。

---

## 3. 免责

本声明的目的是**如实说明来源**，不代表本包已就上述第三方内容取得完整授权。若你是某项资产的权利人并认为本仓库的分发不当，请通过仓库 issue 联系，我们会及时处理。

对本包改编自 Superpowers 的部分，适用其 MIT 许可；对其余第三方资产（品牌规范库、模板），适用上文各自的限制。

