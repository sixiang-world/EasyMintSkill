# Writing Good Tests

**Load this reference when:** 写或改测试、加 mock、为测试加清理/辅助方法时加载。

## Overview

一个测试存在的意义是拦住一个**具体的破坏**。两条原则统管这里的一切：

```
1. Every test names the break it catches
2. Every test exercises the real thing
```

严格的 TDD 会自然产出这两点：先写、且对着真实代码看过它失败的测试，已经证明了自己能失败；只有当局部的真实依赖被证明慢或属于外部时才配得上一个 mock。

## Principle 1: Name the Break

写测试体之前先回答：**哪个生产代码改动会让这个测试失败——那个改动是 bug，还是一个决策？**

一个测试靠"能拦住错误分支、漏掉的副作用、错误的参数、边界情况、被破坏的契约"来赢得自己的位置。

**独立推导期望值。** 用字面量和手算过的固定数据；表驱动测试 + 字面量 `want` 值是首选形态。由被测代码（或它的辅助函数）算出来的期望值，无论被测代码做什么都会通过：

```typescript
// ❌ Mirror assertion: the same builder computes both sides — always true
const expected = buildSearchQuery({ tag: 'urgent' });
expect(buildSearchQuery({ tag: 'urgent' })).toBe(expected);

// ✅ Hand-derived literal
expect(buildSearchQuery({ tag: 'urgent' })).toBe('tag:"urgent"');
```

**不要写 change detector（变更探测器）。** 如果只有"有意为之的决策"能让测试失败——某个常量的值、消息的确切措辞、私有结构——那它只会在重新设计时乱叫，真出 bug 时却睡着。测依赖那个决策的**行为**：不是 `expect(MAX_RETRIES).toBe(5)`，而是"失败的调用被重试 5 次，且第 6 次从未发生"。

**测行为，不测文本。** 断言一个脚本 / skill / 配置里"包含某一行确切文字"，只能证明源文件就是源文件。要把脚本跑在受控输入上，断言输出、副作用或退出码。指导 Agent 的文档类产物，由消费方 Agent 的行为来测；给人看的散文根本不配得到测试。

**测你的代码，不测框架。** 测你的代码在自己边界上立下的契约——你注册的路由、你发出的查询、你产出的载荷。上游机制是上游维护者该写的测试（经典反例：断言你的 router 调用了已注册的 handler——那是框架的测试，不是你的）。当上游行为真的让你意外时，写一个很窄的 characterization test 把那个假设记下来即可。同一条边界也适用于你自己代码内部：构造函数、getter、常量、纯转发，只有在它们负责校验、归一化、兜默认、派生、强制约束或产生副作用时才配得到测试——否则去断言第一个依赖它们的、消费方可见的结果。

### Gate Function

```
BEFORE writing the test body:
  Name the production change that would make this test fail.

  Cannot name one            → redesign around an observable behavior
  "The source text changed"  → run the artifact and assert its effects
  Only intentional decisions → change detector; test the behavior
                               that depends on the decision

  Confirm the expected value is derived without the code under test.
  IF it reuses the code's logic or helpers:
    Replace it with a literal or hand-checked fixture
```

## Principle 2: Exercise the Real Thing

**Mock 不配拥有任何断言。** 对 mock 的断言，在 mock 存在时通过、在 mock 缺失时失败——它对被 mock 的组件一无所知。去断言真实组件的行为；如果你检查的对象就是那个 mock，要么取消 mock，要么删掉这条断言。

```typescript
// ✅ Real behavior
expect(screen.getByRole('navigation')).toBeInTheDocument();

// ❌ Mock existence
expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
```

**用户的质疑：** "Are we testing the behavior of a mock?"（我们是在测 mock 的行为吗？）

**在正确的层级 mock。** 替换真实方法之前先弄清它的每一个副作用；只 mock 那个慢的或外部的操作，测试依赖的部分保持真实。不确定时，先对着真实实现跑一遍测试，观察实际需要发生什么。

```typescript
// ❌ The mock swallows the config write that duplicate detection reads
vi.mock('ToolCatalog', () => ({
  discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
}));

// ✅ Mock only the slow server startup; the config write stays real
vi.mock('MCPServerManager');
```

**让替身具体化。** 当参数、调用次数或调用顺序属于契约的一部分时，就断言它们——一个什么都接受的 fake 什么也没验证。给每个分支（成功、错误、畸形输入）各自独立的固定数据或 spy，这样错误分支无法满足期望。

**完整镜像真实数据。** 按现实中真实存在的结构来 mock——带上全部有文档的字段——而不是只带你测试读到的那些。残缺的 mock 会在下游代码读取被省略字段时静默失败：测试过了，集成挂了。

**生产类只带生产方法。** 只有测试需要的清理逻辑放在测试工具里，绝不作为生产类上的一个 `destroy()`。自问：这方法只有测试调用吗？这个类拥有这个资源的生命周期吗？答案是否 → 放测试工具。

**优先用真实组件而不是复杂 mock。** 当 mock 的准备代码长过测试逻辑本身、mock 缺少真实组件有的方法、或者 mock 一变测试就挂时，换成用真实组件的集成测试。**用户的追问：** "Do we need to be using a mock here?"

### Gate Function

```
BEFORE adding a mock or test helper:
  List the real method's side effects; keep the ones the test
  depends on real — mock the slow/external level below them.

  Mock responses mirror the complete real structure.

  A method only tests call lives in test utilities, not production.

  About to assert on the mock itself?
    Unmock it or delete the assertion.
```

## Tests Ship With the Implementation

TDD 循环——失败测试、最小实现、重构——就是"完成"的定义。把行为需要的测试随实现一起交付，且只要这些：平凡代码和给人看的散文不配得到测试，而为了走流程写出来的测试要付一辈子的维护成本。

## The Mutation Check

收尾之前，在脑子里对生产代码做变异；每一个现实可行的变异都至少应该让一个测试失败：

- 错误的常量或参数
- 错误的分支处理
- 缺失的状态变更或副作用
- 空返回或默认返回
- 对零值、空值、nil、未授权或畸形输入缺少校验

没有任何测试能拦住的变异，说明那段行为是无保护的——或者那个测试是同义反复。

## Quick Reference

| When you... | Do |
|-------------|-----|
| Write any test | Name the break it catches — a bug, not a decision |
| Build an expected value | Derive it by hand; never with the code under test |
| Test a script or document | Run it / pressure-test its consumer; never grep its text |
| Reach for a dependency test | Test your boundary contract, not their documented mechanics |
| Want to assert on a mocked element | Test the real component, or unmock it |
| Are about to mock a method | Learn its side effects; mock the slow/external level |
| Build a mock response | Mirror the real structure completely |
| Need cleanup only tests use | Put it in test utilities |
| Watch mock setup balloon | Switch to an integration test with real components |
| Finish a test file | Run the mutation check |

> 中文对照：写任何测试 → 说出它能拦住的破坏（是 bug，不是决策）；构造期望值 → 手工推导，绝不用被测代码算；测脚本或文档 → 跑它 / 压力测它的消费方，绝不 grep 文本；想测依赖 → 测你自己的边界契约，不测别人有文档的机制；想断言 mock 元素 → 测真实组件或取消 mock；准备 mock 某方法 → 先弄清副作用，只 mock 慢的/外部的层级；构造 mock 响应 → 完整镜像真实结构；只有测试用的清理 → 放测试工具；mock 代码膨胀 → 换成真实组件的集成测试；写完测试文件 → 做变异检查。

## Warning Signs

- Setup 和 assertion 共用同一个对象，保证了二者恒等
- 这个测试只能通过 panic、崩溃或选择器缺失来失败
- 这个测试在每次有意改动时都失败，却从不因意外破坏而失败
- 期望值藏在循环、builder 或辅助函数后面
- 测试去 grep 源码文本，或者断言"某符号保持被删除状态"
- 只留下框架时，这个测试仍然"有意义"
- 为了覆盖率而存在，不检查任何副作用或结果
- 断言检查的是 `*-mock` 这种 test ID，或者删掉 mock 它就失败
- 某个方法只被测试文件调用
- mock 准备代码占测试一半以上，或者你说不清为什么需要这个 mock
- "为了保险起见"而 mock
