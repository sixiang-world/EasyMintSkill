# Defense-in-Depth Validation

## Overview

修完一个由非法数据导致的 bug 之后，只在一处加校验会让人感觉"够了"。但那一处检查会被其他代码路径、重构、mock 绕过。

**Core principle:** Validate at EVERY layer data passes through. Make the bug structurally impossible.
（核心原则：数据流经的每一层都校验。让这个 bug 在结构上不可能发生。）

## Why Multiple Layers

单层校验：「我们修好了这个 bug」
多层校验：「我们让这个 bug 不可能发生」

不同层拦不同的情况：

- 入口校验拦住大部分 bug
- 业务逻辑拦住边界情况
- 环境守卫拦住特定上下文下的危险操作
- 调试埋点在别的层都失效时提供线索

## The Four Layers

### Layer 1: Entry Point Validation

**目的：** 在 API 边界拒绝明显非法的输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### Layer 2: Business Logic Validation

**目的：** 保证数据对这个操作而言是合理的

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### Layer 3: Environment Guards

**目的：** 阻止在特定上下文里执行危险操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### Layer 4: Debug Instrumentation

**目的：** 为事后取证保留上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## Applying the Pattern

当你发现一个 bug：

1. **追踪数据流** —— 坏值从哪来？在哪被用？
2. **列出所有检查点** —— 数据流经的每一个点
3. **每层都加校验** —— 入口、业务、环境、调试
4. **逐层测试** —— 试着绕过第 1 层，验证第 2 层能拦住

## Example from Session

**Bug：** 空的 `projectDir` 导致 `git init` 跑在源码目录里

**数据流：**
1. 测试 setup → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 里运行

**加的四层：**
- 第 1 层：`Project.create()` 校验非空/存在/可写
- 第 2 层：`WorkspaceManager` 校验 projectDir 非空
- 第 3 层：`WorktreeManager` 在测试环境拒绝在 tmpdir 之外 git init
- 第 4 层：git init 之前打印堆栈日志

**结果：** 1847 个测试全过，bug 无法复现

## Key Insight

四层缺一不可。在测试过程中，每一层都拦住了别的层漏掉的 bug：

- 不同的代码路径绕过了入口校验
- mock 绕过了业务逻辑检查
- 不同平台上的边界情况需要环境守卫
- 调试日志识别出了结构性误用

**Don't stop at one validation point.** 每一层都加检查。
