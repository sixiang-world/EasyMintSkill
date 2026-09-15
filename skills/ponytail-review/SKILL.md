---
name: ponytail-review
description: >
  Code review focused exclusively on over-engineering. Finds what to delete:
  reinvented standard library, unneeded dependencies, speculative abstractions,
  dead flexibility. One line per finding: location, what to cut, what replaces
  it. Use when the user says "review for over-engineering", "what can we
  delete", "is this over-engineered", "simplify review", or invokes
  /ponytail-review. Complements correctness-focused review, this one only
  hunts complexity.
---

Review diffs for unnecessary complexity. One line per finding: location, what
to cut, what replaces it. The diff's best outcome is getting shorter.

## Format

`L<line>: <tag> <what>. <replacement>.`, or `<file>:L<line>: ...` for
multi-file diffs.

Tags:

- `delete:` dead code, unused flexibility, speculative feature. Replacement: nothing.
- `stdlib:` hand-rolled thing the standard library ships. Name the function.
- `native:` dependency or code doing what the platform already does. Name the feature.
- `yagni:` abstraction with one implementation, config nobody sets, layer with one caller.
- `shrink:` same logic, fewer lines. Show the shorter form.

## Examples

❌ "This EmailValidator class might be more complex than necessary, have you
considered whether all these validation rules are needed at this stage?"

✅ `L12-38: stdlib: 27-line validator class. "@" in email, 1 line, real validation is the confirmation mail.`

✅ `L4: native: moment.js imported for one format call. Intl.DateTimeFormat, 0 deps.`

✅ `repo.py:L88: yagni: AbstractRepository with one implementation. Inline it until a second one exists.`

✅ `L52-71: delete: retry wrapper around an idempotent local call. Nothing replaces it.`

✅ `L30-44: shrink: manual loop builds dict. dict(zip(keys, values)), 1 line.`

## Scoring

End with the only metric that matters: `net: -<N> lines possible.`

If there is nothing to cut, say `Lean already. Ship.` and stop.

## Boundaries

Complexity only, correctness bugs, security holes, and performance go to a
normal review pass, not this one. A single smoke test or `assert`-based
self-check is the ponytail minimum, not bloat, never flag it for deletion.
Does not apply the fixes, only lists them.
"stop ponytail-review" or "normal mode": revert to verbose review style.

## Red Flags — 出现这些念头就停下

> 这些念头说明你正在给自己找借口。它们的出现本身就是信号。

- 这不算复杂吧，挺标准的写法
- 作者大概有他的理由，我不多嘴了
- 提出这么多删改意见，会不会显得太苛刻
- 这层抽象虽然只有一个实现，但留着也不碍事
- 只有一个文件，没什么可审的
- 这行是我自己写的，跳过
- 别的地方都用这个模式，不该单挑出来
- 说"这里可能过度设计"比直接说删哪儿更委婉

**All of these mean: 逐行找可删之处，一行一条，写出位置 + 删什么 + 用什么替代，结尾给 `net: -<N> lines`。**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "This isn't really complex" | Complexity is not a feeling, it is a count. One implementation, one caller, one product — that is the definition. |
| "The author probably had a reason" | Reasons belong in the code or the review. Unexplained complexity is the finding. |
| "Listing all these cuts will look harsh" | The diff's best outcome is getting shorter. Being nice about bloat is how bloat survives. |
| "One implementation, but the abstraction doesn't hurt" | It hurts the next reader and the next change. Inline it until a second implementation exists. |
| "It's a single file, nothing to review" | File count is irrelevant. A 400-line file is where the biggest cuts usually live. |
| "I wrote this part, skip it" | Your own code is where you are blindest. Review it hardest. |
| "The rest of the codebase does it this way" | A widespread pattern is a bigger finding, not an exemption. |
| "I'll phrase it gently: 'might be more complex than necessary'" | Hedge language gets ignored. `L12-38: stdlib: 27-line validator class. "@" in email, 1 line.` |

> 中文说明：这份 review 的产出是「可删除清单」，不是委婉建议——一行一条、给出位置与替代方案，结尾 `net: -<N> lines possible.`。没有可删的就直说 `Lean already. Ship.` 并停下。只找复杂度，正确性/安全/性能不归这里。
