---
name: project-run
description: >-
  运行配置。用户问「怎么启动项目」「加个运行方式」「新增/修改运行命令」、
  写常用脚本（运行/构建/打包/安装/发版部署等），或项目完成需要生成运行配置时使用。
  提供 .agentskill/run.json 的完整格式规范。
---

# Project Run — 运行配置

`.agentskill/run.json` 由运行配置面板读取，每条 commands 显示为一个可一键启动/停止的按钮（含端口状态）。文件变化时配置自动刷新。

## 格式

```json
{
  "commands": [
    { "platform": "react", "label": "前端", "cwd": "./client", "run_command": "npm run dev", "url": "http://localhost:5173" },
    { "platform": "spring", "label": "后端", "cwd": "./server", "run_command": "mvn spring-boot:run", "url": "http://localhost:8080" }
  ]
}
```

## 字段

- **platform**：**按 run_command 判断（解释器+文件扩展名或首个命令词），不要按项目技术栈硬标**。合法值：react/vue/nextjs/nuxt/angular/svelte/spring/django/flask/fastapi/nodejs/rails/laravel/go/rust/dotnet/react-native/expo/flutter/electron/tauri/python/shell/git/java。对应关系：flutter 开头→flutter、git 开头→git、npm/pnpm/yarn/node→nodejs、python→python、bash/sh/./xx.sh→shell、mvn/gradle/java→java、cargo→rust、go→go、dotnet→dotnet、docker→shell；脚本文件按扩展名（.py→python、.sh→shell、.js/.ts→nodejs、.java→java、.rb→rails、.go→go、.rs→rust、.dart→flutter）。mac 桌面脚本用 shell，桌面应用用 electron。（配置显示时也会按此规则推断，写对避免跳变）
- **label**：显示名，如"前端"、"后端"、"Android"（命名规则见下方「脚本标题命名」）
- **cwd**：工作目录（相对项目根），默认 "."
- **run_command**：运行命令，如 npm run dev、python main.py、flutter run、flutter build apk、bash deploy.sh
- **install_command**（可选）：依赖安装命令，如 flutter pub get、npm install。保留记录用，配置面板当前仅展示不提供安装按钮
- **url**（可选）：启动后访问地址，如 http://localhost:3000（仅运行类脚本需要，构建/打包/安装类不填）

## 规则

- 多入口（前后端分离、跨平台）写多条
- **静态 HTML 页面（纯 HTML/CSS/JS，无构建工具、无 npm 依赖）不需要开发服务器**：run_command 用 open index.html（mac 直接打开）或 python3 -m http.server 8000（本地静态托管），url 对应 file:// 路径或 http://localhost:8000。不要为静态页面无谓创建 dev server 或引入框架构建
- 用户提出相关需求时，直接读现有 run.json 追加/更新——不必等任务全部完成，项目可运行即可生成；项目完成时生成（每次回到 done 更新）

## 脚本管理（不只限于启动项目）

- **脚本形式分工**：
  - **简单单行命令**（构建/打包/运行等一步到位）→ 直接写在 run_command：`flutter build apk`、`npm run build`
  - **复杂多步脚本**（条件/循环/多命令串联，如发版部署）→ 先写脚本文件到 `scripts/` 下（如 `scripts/release.sh`，加执行说明注释），run_command 写 `bash scripts/release.sh` 引用路径——脚本可版本管理、可独立测试、可复用
  - 配置编辑内 run_command 可直接改，简单命令无需单独建文件
- **脚本内不要手动重定向输出**（如 `> file 2>&1`、`| tee`）：运行时会自动收集 stdout/stderr（日志面板实时显示 + 落盘），重定向会绕过收集导致面板无输出（与 bash 工具规则一致）

run.json 不只放「启动项目」命令——**用户日常反复使用的脚本都加入管理**：运行、构建、打包、安装依赖、git 发版部署、数据备份、测试等。用户说「帮我写个 xx 脚本」时：

- 写完后**询问用户**「是否加入运行配置？」；若判断脚本属于**长期反复使用**（如打包/构建/部署），**默认添加**并在回复中告知「已加入运行配置」
- 用户没有明确要求写脚本但场景需要（如「我要打包发给朋友」）时，主动写对应脚本并加入配置面板
- 已配置脚本可被用户编辑（点击标题）或删除（删除需确认）——不要擅自删除用户配置里的脚本

### 预设脚本（项目完成时生成）

项目开发完成回到 done 时，按项目技术栈生成常用预设脚本（已有则跳过）：

- **运行**：本地开发启动（如 flutter run / npm run dev）
- **构建/打包**：对应平台的构建产物命令（如 flutter build apk / flutter build macos / npm run build）
- **安装**：产物安装命令（如 mac 的 open/安装到 Applications）
- **发版部署**（如项目有 git 发版流程时）：版本号更新 + 打包 + 提交 + tag + 推送的命令链

### 脚本标题命名

**标题必须简单准确、一目了然**——用户看标题就知道脚本干什么。规则：

- 用「动作 + 对象」式短名称，避免口语化，避免技术黑话
- 平台环境差异写进标题（同一动作多平台时）
- 示例：
  - flutter 项目 mac 环境打包安卓 → `安卓端打包`
  - 构建 iOS 并安装到真机 → `安装iOS Release版本`
  - 运行 mac 桌面端 → `本地dev运行`
  - 构建 macOS 产物并安装到 Applications → `安装到本地`
  - 前端项目启动 dev server → `前端dev运行`
- 同一动作只有一个平台时不需要写平台名（如 `打包` 即可）；多平台才加平台区分

---

## 🔴 CHECKPOINT · 脚本确认

- **新增脚本到运行配置**：写完脚本后必须询问用户「是否添加到运行配置？」；若判断属于长期反复使用（打包/构建/部署），默认添加并告知
- **删除脚本**：删除运行配置中的脚本必须确认，不能擅自删除用户配置
- **发版部署脚本**：涉及 git push/tag/发布的脚本必须用户明确授权后才能执行，不能自动推送

## 失败模式与恢复

| 场景 | 触发条件 | 恢复动作 |
|---|---|---|
| run.json 格式错误 | JSON 语法错误导致面板无法读取 | 备份当前文件 → 修复 JSON 语法 → 验证可解析；不覆盖用户已有配置 |
| 脚本命令执行失败 | run_command 报错或退出码非 0 | 记录具体错误 → 告知用户 → 让用户决定重试/修改/跳过；不静默失败 |
| platform 字段标注错误 | platform 与 run_command 不匹配导致显示跳变 | 按 run_command 重新判断 platform（解释器+扩展名或首个命令词），不按项目技术栈硬标 |
| 端口冲突 | 启动的服务端口被占用 | 提示用户端口冲突，给出换端口或停止占用进程的选项；不自动 kill 其他进程 |

## 反例黑名单（不要做以下事情）

- **不要为静态 HTML 页面创建 dev server**：纯 HTML/CSS/JS 无构建工具的项目直接用 `open index.html` 或 `python3 -m http.server`，不引入框架构建
- **不要在脚本内手动重定向输出**：`> file 2>&1`、`| tee`、`nohup ... &` 会绕过运行时的输出收集，导致日志面板无内容
- **不要把简单单行命令写成脚本文件**：构建/打包/运行等一步到位的命令直接写在 run_command，不必要的脚本文件增加维护成本
- **不要擅自删除用户运行配置**：面板内脚本可被用户编辑或删除，AI 不擅自删除——删除需确认
- **不要把 platform 按项目技术栈硬标**：platform 按 run_command 判断（解释器+扩展名或首个命令词），不按「这是 React 项目所以标 react」
