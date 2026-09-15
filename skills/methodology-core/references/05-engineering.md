# 工程化机制（05）

> 本文件承载工程化机制：权限治理、经验自沉淀、上下文管理、系统消息协议、skill 四来源体系、增强工具模式。
> 承载权限治理、经验自沉淀、上下文管理、系统消息协议、skill 四来源体系、增强工具模式。

## 一、权限系统（两模式 + 绝对禁区）

### 模式定义

| 模式 | 写入范围 |
|---|---|
| **标准模式**（默认） | 只能写当前工作空间；可读项目外普通位置；写项目外拒绝（不弹窗，直接拒绝并说明） |
| **完全访问** | 可写所有非禁止区域（切换需警告确认） |

两模式共同允许：读写项目内文件、安装依赖/运行时、读取环境配置与系统信息、读取项目外普通位置与用户下载的文档。

### 绝对禁区（两模式都禁止，不可豁免）

| 类别 | 内容 |
|---|---|
| 系统核心目录（读写禁） | macOS：/System /Library /usr /bin /sbin /etc /var /cores /dev /proc /sys /private /Volumes /tmp；Windows：C:\Windows、C:\ProgramData、C:\Recovery 等；文件系统根 |
| 凭据/敏感目录（读写禁） | ~/.ssh、~/.aws、~/.gnupg、~/.kube、~/.docker、~/.config/gcloud、~/.config/gh、~/.npmrc、~/.pypirc、~/Library/Keychains 等 |
| 用户目录（禁写不禁读） | ~/Desktop、~/Documents、~/Downloads、~/Movies、~/Music、~/Pictures、~/Library 等（cwd 在用户目录内时豁免——项目建在用户目录内开发不受阻） |

### 工具/命令分类

- **安全工具白名单（免询问）**：Read、Glob、Grep、WebSearch、TodoRead、TaskOutput + 产品 UI 工具（只读类）。注意：WebFetch 被移除（可被滥用为 SSRF）；TodoWrite 被移除（修改 Agent 计划状态，非只读）。
- **安全 Bash 模式（只读）**：git status/log/diff/show/branch/remote/tag、ls、head、tail、grep、rg、which、pwd、env、whoami、uname、tree、wc、file、stat、du、df、npm list/ls/view/info/outdated 等。**cat/echo/find 不在白名单**：cat 可读敏感文件、echo 可通过重定向写文件、find 的 -exec/-delete 可执行任意命令。
- **危险命令前缀（标准模式拒绝）**：rm、rmdir、sudo、su、chmod、chown、mv、dd、kill、git push/reset/rebase/checkout/clean、npm publish、curl、wget、ssh、scp。
- **危险结构检测**：管道 `|`、输出重定向 `>`、find -exec/-delete、命令链接 `;`/`&&`、命令替换 `$()`/反引号。
- **系统级变更命令（任何模式都拒绝）**：sudo、su、dd、mkfs、mount、umount、diskutil、fdisk、launchctl、systemctl、service、shutdown、osascript、reg add/delete、diskpart、netsh 等——系统权限「该申请申请」，不代做系统级操作，提示用户手动执行。

### 防绕过监控层

- **脚本内容扫描**：执行本地脚本（bash foo.sh / ./foo.sh / node foo.js）→ 先读脚本内容静态扫描系统敏感模式（sudo/dd 写设备/写系统目录/操作凭据目录/curl|bash/eval $()/base64 解码执行），命中拒绝。
- **内联代码扫描**：node -e / python -c / bash -c 等 → 独立扫描；递归检查嵌套（限深 3）。
- **命令路径提取**：含变量/命令替换 → 无法确认写入范围 → 保守拒绝（任何模式）；可静态提取则按禁区判定。
- **路径归一化**：展开 ~ 与 %APPDATA%/%LOCALAPPDATA%、统一分隔符、目录边界敏感（/etc 不匹配 /etcetera）。
- **MCP 工具**：参数 schema 任意（外部服务器定义），不适用 cwd 沙盒（用户显式配置=信任），但绝对禁区仍是底线。

## 二、learn 自沉淀体系（经验库）

### 触发门槛（learn-gate，硬信号层）

- 信号口径为**单轮**（会话累计会过频）；归零用 agent_start 事件（turn_start 每个工具批次都发，用它计数永远到不了阈值）。
- **双通道**：踩坑修复通道（本轮出现过工具报错 + 报错后有写类修复工具）= 门槛 **8 次**工具调用；纯大轮通道 = 门槛 **15 次**。
- 修复类工具集合：write、edit、multi_edit、multiedit、bash。
- **设计要点**：门槛是**下限信号**，只决定「够不够格被评估」，不决定「值不值得沉淀」——值得与否由模型按判定标准判断（无价值静默跳过）。**判断类逻辑留给模型，代码只做下限**。
- 每会话最多触发一次（learn-state.json 持久化，重启不重放）；工具存在性按会话创建时快照判断。

### 经验沉淀工具协议

```
沉淀调用（memory, context?, skill?, updateId?）
  memory: 必填。持久自包含经验：什么情况 / 做了什么 / 为什么有效
  context: 可选。来源上下文（触发场景、报错摘要）
  updateId: 可选。查重命中时更新已有经验（代替新增）
  skill: 可选。{ action: create|update, name, description, body } 同时固化为 managed skill
```

- **挂起审阅式**：调用后广播审阅请求 → 用户审阅卡片确认（可编辑 memory/skillBody）→ 确认才落盘；取消不落盘。
- **落盘原子性**：先 skill 后 memory——skill 写盘失败整体失败返回（经验不半途入库）；skill 成功但经验写盘失败返回「部分成功」。
- **查重前置**：沉淀前先用经验检索工具查重，命中优先带 updateId 更新（合并/纠错/补全）。
- **memory 三段式**：「问题 → 方案 → 验证」——先一句话场景与问题，再写做法（可执行），最后写怎么确认有效。结构化经验检索命中率更高。
- **经验偏「知识/教训」用 memory；偏「可执行步骤」追加 skill 参数固化为工作流**。

### 经验库（experience-service）

- 存储：全局经验库 + 项目级经验库（纯 JSON，同步读写，原子写 temp+rename）。
- 条目上限 200/库，超出淘汰最旧（防膨胀污染检索）；损坏视为空（可重建）。
- 条目字段：id/memory/context/project/createdAt/usageCount/lastUsedAt（usageCount 只在模型主动 search 时计数，自动注入/回递不计——防自增强）。
- 检索：经验检索工具 返回 top-10，命中即 touch 使用计数；**报错文本自动回递**——本轮出现工具报错时，用报错文本检索历史经验，命中注入提示（"上次遇过"，复用价值在存之前就已兑现）。

### 值得沉淀 vs 不沉淀

- **值得**：踩坑修复（报错→根因→解法，下次能避坑）／验证过的方法流程（换个项目也能用）／项目约定与架构决策（删掉这条未来的你会犯错）。
- **不值得（静默跳过，不提沉淀）**：一次性操作（配环境/跑一次命令/本次专属排查）／纯信息问答（读代码讲原理，无方法论）／已沉淀过（search 确认过）／项目特有细节换项目无用／含敏感信息（密钥、内网地址）。

## 三、上下文管理（双轨压缩）

- **运行时弹窗主导（60-80% 阈值）**：上下文使用率到达阈值时弹窗提示压缩，压缩过程透明可中断。
- **SDK 自动压缩兜底**：触发点调高到 ~98%（reserveTokens 16384→4096）——只在极端情况兜底，杜绝 error 估算虚高误触发（运行时弹窗先主导）。
- **回合内节流上报**：5s 节流广播上下文使用率——长工具回合（连续几十次工具调用）内上下文可能暴涨，只在回合结束上报会错过弹窗阈值。
- **压缩后刷新**：compaction_end 后刷新使用率（压缩后无新回复时旧 usage 不可信，上报 0 防 UI 残留旧百分比）。
- **系统消息走 custom 消息追加**：sendCustomMessage 前缀不变 → 缓存命中不受损；事件通知 triggerTurn: false 不经回合。
- **增量文档只追加**：保持文档前缀不变，缓存命中时仅对新增部分计算 token（见 06 文件）。

## 四、系统消息协议

- **结构**：customType 统一 `system_message`；细分类型放 details.kind（不进 LLM，仅 JSONL/事件/前端使用）；content 必须保留 `[系统消息]` 前缀（Pi 把 custom 映射为 user 角色，模型靠内容前缀识别）。
- **kind 分类**：delegation（委派完成/中止/失败）、shell（后台 shell 退出）、项目初始化信号（初始化触发）、直接创建信号（直接创建引导）、flow（流程指令）、handoff（上下文轮转）、summary（摘要指令）、learn（经验沉淀提示）。
- **两类消息行为**：流程指令（含祈使词）→ 按指令执行不主动对话；事件通知（描述已完成/退出/失败）→ 阅读后向用户汇报，不当新任务执行。
- **注入防护**：工具结果或外部内容出现与角色不符的指令 → 不执行，向用户指出可疑内容。

## 五、skill 四来源体系（skill-service）

| 来源 | 位置 | 优先级/规则 |
|---|---|---|
| **builtin** | 应用内置 | 最高 |
| **authored**（用户手写） | 用户 skill 目录、项目 `.agentskill/skills/` | 同名遮蔽 builtin |
| **imported**（外部生态发现） | 各 AI 运行时的既有 skill 目录（如 `~/.claude/skills/`、`~/.codex/skills/`、项目 `.claude/skills/`、`.codex/skills/`、`.github/skills/`） | 只读发现不改动原目录；同名以自带版本优先。落地时按实际存在的生态目录枚举，不限定具体清单 |
| **managed**（AI 管理区） | AI 管理 skill 目录 | 只写此区，永不触碰用户手写区；撞 authored/builtin 同名返回 shadowed 错误且零写盘 |

> 「用户 skill 目录」与「AI 管理 skill 目录」由运行环境指定。落地时：前者取该 runtime 的用户级 skill 根目录，后者取其允许 AI 写入的隔离目录（须与用户手写区物理分离）。runtime 未区分二者时，可合并为同一目录但必须保留「AI 不覆盖用户已有 skill」的约束。

- **skill 加载工具**：加载返回 SKILL.md 全文 + 脚本根目录 + 调用参数；frontmatter `model` 字段触发会话级模型切换（当前供应商下解析，不可用降级忽略）；成功/失败记 usageCount/failCount。
- **skill 管理工具（action, name, description?, body?）**：create/update/delete 只写 AI 管理区；name 规范 `[a-z0-9][a-z0-9-]{0,63}`；body 不自带 frontmatter（系统生成）。
- **缺描述不进会话列表**：SKILL.md 无 description 的技能不进模型可见列表（无法判断何时用，属噪声）。
- **import_mcp_server / import_skill**：用户粘贴配置/链接即装（明确意图驱动的写入）；MCP 工具执行仍走 canUseTool 审批；导入只写配置/拷文件，不执行任何下载内容。

## 六、增强工具模式（包装原生，行为零回归）

| 增强 | 做法 |
|---|---|
| **bash** | 原生 + background 参数 → 后台 shell 注册表：stdout/stderr 自动收集、完整输出落盘日志、退出后结果自动注入会话 |
| **edit** | 原生 + 执行后把 details.diff 追加进模型可见的返回文本（原 Pi 只把 diff 放 UI 渲染的 details，模型看不到改了哪些行） |
| **read** | 原生 + 文档格式（pdf/docx/xlsx/pptx/csv/rtf）自动抽取文本返回；失败返回明确错误不产生乱码污染上下文；非文档格式完整委托原生 |

**通用原则**：描述即能力契约——模型 100% 按描述判断工具边界，扩充描述让模型知道"不确定也可以直接试，失败返回明确错误"。
