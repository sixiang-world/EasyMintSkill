# 集成层（可选）

本目录提供**可选**的宿主平台接入脚本，用于解决一个自举问题：

> 技能包里的 `SKILL.md` 是**惰性文件** —— 放在磁盘上，不会自己生效。
> 没有注入层，入口技能就得靠「模型自己想起来去查」，
> 而入口技能存在的意义恰恰是**强制触发**。

**不装这一层，技能包依然可用** —— 但你需要知道代价（见下方「前提声明」）。

---

## 一、前提声明（重要）

本包是**平台中立**的，不绑定任何特定运行时。这带来一个必须讲清楚的前提：

| 你的宿主平台 | 入口技能能否自动触发 |
|---|---|
| 具备 skill 自动发现/加载能力（多数现代 Agent 运行时） | ✅ 能 —— 无需做任何事 |
| 只能在用户**显式点名**时才加载 skill | ⚠️ **不能** —— `using-methodology` 的强制触发不成立 |
| 完全没有 skill 机制，只有系统提示词 | ⚠️ **不能** —— 需要你自己把入口技能贴进系统提示词 |

**第二、三种情况下，本包的「两层强制触发」设计会退化为「建议」。** 这时你有三个选择：

1. **接入本目录的注入器**（推荐，见下）
2. **手工引导**：在每个新会话开头对 Agent 说一句「先读 `using-methodology`，然后再回应我」
3. **把入口技能写进你的系统提示词 / 项目说明文件**：
   ```
   任何回应或动作之前，先调用相关 skill（含 1% 可能）。
   创作性任务先经 kickoff 分类规模、澄清需求、拿到我的批准，再动手。
   ```

**我们选择如实告知，而不是假装触发一定会发生。** 若你发现 Agent 没走流程直接动手，可以直接提醒它先查 skill。

---

## 二、`session-start` 注入器

在会话启动时，把入口技能 `using-methodology` 的**全文**注入到 Agent 的上下文里 —— 让"先查 skill"这条规则在它看到你的消息之前就已经在它的上下文里。

### 用法

```bash
./session-start --format <shape> [--skills-dir <path>]
```

| 参数 | 说明 |
|---|---|
| `--format` | **必填**。输出形状（见下表）。不猜环境，由你显式指定。 |
| `--skills-dir` | 可选。技能包根目录。默认从脚本位置向上推一级。 |

### 输出形状

| `--format` | 输出的 JSON | 适用 |
|---|---|---|
| `nested` | `{"hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":"..."}}` | 消费嵌套 hook 输出的运行时 |
| `snake` | `{"additional_context":"..."}` | 消费蛇形字段的运行时 |
| `sdk` | `{"additionalContext":"..."}` | 消费顶层字段的 SDK 标准形状 |

**为什么格式必须显式指定，而不是靠环境变量自动判定？**
某些运行时会**同时**读取嵌套与顶层字段且**不去重** —— 自动判定一旦判错，同一条上下文会被注入两遍，每轮对话都白白烧掉一份 token。让调用方显式声明，比让脚本猜更安全。

### 自测

```bash
# 看三种形状是否都能正常输出
./session-start --format sdk   | head -c 200
./session-start --format nested | head -c 200
./session-start --format snake | head -c 200

# 校验 JSON 合法
./session-start --format sdk | python -m json.tool > /dev/null && echo OK
```

### 接入方式（按你的平台选一条）

**方式 A：平台有 hook 机制**
在平台的 hook 配置里注册 `SessionStart`（或等价的会话启动事件），命令指向本脚本，并**按平台文档确定 `--format` 取值**。

```jsonc
// 示例骨架 —— 字段名请以你所用平台的文档为准
{
  "hooks": {
    "SessionStart": [
      { "command": "bash /path/to/integrations/session-start --format <shape> --skills-dir /path/to/pack" }
    ]
  }
}
```

**方式 B：平台没有 hook，但你能控制系统提示词**
把注入器的输出拼进系统提示词：

```bash
PACK=/path/to/pack
CONTEXT=$(bash "$PACK/integrations/session-start" --format sdk --skills-dir "$PACK" \
          | python -c 'import json,sys; print(json.load(sys.stdin)["additionalContext"])')
```

**方式 C：什么都没有**
用第一节的「选项 2」手工引导，或「选项 3」把规则写进说明文件。

---

## 三、接入时的硬约束（踩过的坑）

若你为某个平台写自己的注入器，这几条是**必须遵守**的：

| 约束 | 原因 |
|---|---|
| **注入为 user 角色消息，不要注入 system** | 注入 system 会在每一轮对话里重复，token 成本爆炸；且多个 system 消息会破坏部分模型的行为 |
| **必须有去重守卫（dedup guard）** | 部分平台的回调会重复触发，没有守卫就会注入多次 |
| **上下文压缩（compaction）之后必须重注入** | 压缩会把注入内容一起压掉，不重注入则规则失效 |
| **`<EXTREMELY_IMPORTANT>` 标记可兼作去重哨兵** | 用同一个标记既做强调、又做"是否已注入"的检测，省一次状态存储 |
| **不要把三种输出形状混发** | 见上文「为什么格式必须显式指定」 |

---

## 四、与技能包的关系

```
宿主平台
  └→ [本目录的注入器]  ← 可选，解决"入口技能怎么被触发"
       └→ 把 using-methodology 注入上下文
            └→ using-methodology（调度层：判断用哪个 skill）
                 └→ kickoff（创作起手层）
                      └→ ... 后续技能
```

**注入器本身不含任何方法论**，它只负责把入口技能送进上下文。方法论全在 `skills/` 里。
