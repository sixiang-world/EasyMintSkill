# 品牌设计规范库

本目录收录 74 个知名品牌的设计系统分析，每个品牌一个子目录，内含 `DESIGN.md`。

## 目录结构

```
brand-tokens/
├── stripe/
│   └── DESIGN.md
├── vercel/
│   └── DESIGN.md
└── ...（共 74 个）
```

## DESIGN.md 格式

YAML frontmatter + Markdown 正文。frontmatter 是可解析的 token 定义，正文是设计语言解读。

```yaml
---
version: alpha
name: <品牌>-Inspired-design-analysis
description: <设计语言概述>
colors:      # 语义色 → 十六进制
typography:  # 字号 token → fontFamily/size/weight/lineHeight/letterSpacing
rounded:     # 圆角梯度
spacing:     # 间距梯度
components:  # 组件 token → 引用上面的 color/typography/rounded
---
```

正文包含 Overview / Colors / Typography / Layout / Elevation / Shapes / Components / Do's and Don'ts / Responsive 等章节，引用 token 时用 `{colors.primary}` 这类花括号语法。

## 使用方式

1. 用户提到某品牌风格时，`Read` 对应目录的 `DESIGN.md`。
2. 解析 frontmatter 取色板与排版 token。
3. 把 token 值写进页面的 CSS 变量，不要照抄品牌名或商标。

## 授权与来源

见 `../THIRD_PARTY_NOTICES.md`。**使用时请只取设计语言（配色、排版节奏、圆角、间距体系）作为风格参考，不要复制品牌名称、商标、logo 或专有字体文件。**
