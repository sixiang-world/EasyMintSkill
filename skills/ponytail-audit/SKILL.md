---
name: ponytail-audit
description: >
  Whole-repo audit for over-engineering. Like ponytail-review, but scans the
  entire codebase instead of a diff: a ranked list of what to delete, simplify,
  or replace with stdlib/native equivalents. Use when the user says "audit this
  codebase", "audit for over-engineering", "what can I delete from this repo",
  "find bloat", "ponytail-audit", or "/ponytail-audit". One-shot report, does
  not apply fixes.
---

ponytail-review, repo-wide. Scan the whole tree instead of a diff. Rank
findings biggest cut first.

## Tags

Same as ponytail-review:

- `delete:` dead code, unused flexibility, speculative feature. Replacement: nothing.
- `stdlib:` hand-rolled thing the standard library ships. Name the function.
- `native:` dependency or code doing what the platform already does. Name the feature.
- `yagni:` abstraction with one implementation, config nobody sets, layer with one caller.
- `shrink:` same logic, fewer lines. Show the shorter form.

## Hunt

Deps the stdlib or platform already ships, single-implementation interfaces,
factories with one product, wrappers that only delegate, files exporting one
thing, dead flags and config, hand-rolled stdlib.

## Output

One line per finding, ranked: `<tag> <what to cut>. <replacement>. [path]`.
End with `net: -<N> lines, -<M> deps possible.` Nothing to cut: `Lean already. Ship.`

## Boundaries

Complexity only, correctness bugs, security holes, and performance go to a
normal review pass. Lists findings, applies nothing. One-shot.
"stop ponytail-audit" or "normal mode" to revert.

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 仓库太大了，扫几个核心目录就够
- 这个依赖虽然只用了一个函数，但换掉风险大
- 找不到引用就先留着，宁可少提一条
- 这文件这么整齐，不用细看
- 有几个发现就够了，不用穷尽
- 顺手把这些问题改掉，比只列清单更有用
- 每个发现都要写清理由，太长的报告不合适

**All of these mean: 全树扫描、按「删得最多」排序、一行一条给替代方案，结尾 `net: -<N> lines, -<M> deps possible.`**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "The repo is huge, scanning a few core dirs is enough" | This is the whole-tree version of the review. Unscanned dirs are where the untouched bloat sits. |
| "Only one call site, but swapping the dep is risky" | Risk is why you list it instead of applying it. Name the replacement and let the user judge. |
| "No references found, so I'll leave it alone" | Zero references is the finding. Report it as `delete:` with nothing as the replacement. |
| "This file looks tidy, no need to read closely" | Tidy formatting hides single-implementation interfaces better than messy code does. |
| "A few findings are enough, no need to be exhaustive" | A ranked list that stops early is a partial answer. Scan, then rank — don't stop at the first hits. |
| "Might as well fix them while I'm here" | This is a one-shot report. Applying fixes mixes the audit with a refactor and destroys the ranking. |
| "A long report is bad form" | One line per finding is the format. Long reports happen because the repo is long. |

> 中文说明：这是 ponytail-review 的全仓库版——一次性的只读报告，排序依据是「能删掉多少」。不要顺手改代码，也不要因为文件整齐或目录陌生就跳过；找不到引用的死代码本身就是最有价值的一条发现。
