> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: pdf-generator
description: 使用 Quarto/Typst 技术栈与现代设计模板生成专业 PDF
effort: low
version: 1.0.0
---

# PDF 生成器技能

使用 Quarto + Typst 生成具有现代排版风格的专业 PDF。

## 技能用途

本技能可协助完成以下任务：
- 搭建 Quarto/Typst 项目
- 创建文档模板
- 从 Markdown 生成 PDF
- 排查渲染问题
- 自定义设计系统

## 技术栈

| 工具 | 版本 | 职责 |
|------|---------|------|
| **Quarto** | ≥1.4.0 | 文档渲染引擎 |
| **Typst** | 0.13.0 | 现代排版（内置集成） |
| **Pandoc** | 3.x | Markdown 转换（内置集成） |

### 生成流水线

```
  源文件              工具            模板                  输出
  ──────              ─────           ────────              ──────

  .qmd  ──────────► Quarto ────► --to whitepaper-typst ──► Typst 0.13 ──► .pdf ✅
  (Markdown           │           (_extensions/               (~270K–1.7M,
  + YAML)             │            typst-template.typ)         有样式)
                      │
                      └──────► --to epub ──► Pandoc ──────────────────► .epub
                                             + epub-styles.css

  ⚠️  --to pdf（无模板）→ PDF 体积小、无样式 → 始终优先使用 --to whitepaper-typst
```

### 可用格式

```
  ┌──────────────────────┬────────────────────────┬──────────────────┐
  │ 格式                 │ 命令                   │ 输出             │
  ├──────────────────────┼────────────────────────┼──────────────────┤
  │ 有样式 PDF ✅        │ --to whitepaper-typst  │ ~270K–1.7M       │
  │ 标准 PDF ❌          │ --to pdf               │ ~80-190K，原始   │
  │ EPUB                 │ --to epub              │ epub-output/     │
  └──────────────────────┴────────────────────────┴──────────────────┘
```

## 快速开始

### 安装

```bash
# macOS
brew install quarto

# Linux
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.4.555/quarto-1.4.555-linux-amd64.deb
sudo dpkg -i quarto-1.4.555-linux-amd64.deb

# Windows
winget install Posit.Quarto
```

### 生成 PDF

```bash
# 单个文件
quarto render document.qmd

# 所有文件
quarto render *.qmd

# 热重载预览
quarto preview document.qmd
```

## YAML Frontmatter 模板

```yaml
---
title: "文档标题"
subtitle: "可选副标题"
author: "作者姓名"
date: 2026-01-17
date-format: "MMMM YYYY"
format:
  typst:
    toc: true
    toc-depth: 2
    section-numbering: "1.1"
lang: en
---
```

### 可用参数

| 参数 | 类型 | 说明 |
|-----------|------|-------------|
| `title` | string | 主标题（封面页） |
| `subtitle` | string | 可选副标题 |
| `author` | string | 作者 |
| `date` | date | ISO 格式（YYYY-MM-DD） |
| `date-format` | string | 显示格式（`MMMM YYYY`） |
| `toc` | boolean | 是否显示目录 |
| `toc-depth` | number | 目录深度（1-3） |
| `section-numbering` | string | 编号格式（`1.1`、`1.a`） |
| `lang` | string | 语言（`fr`、`en`） |

## 项目结构

```
project/
├── _extensions/
│   └── custom-template/
│       ├── _extension.yml      # 扩展元数据
│       ├── typst-template.typ  # 主模板
│       └── typst-show.typ      # Quarto → Typst 桥接
├── document.qmd                # 源文件
└── document.pdf                # 生成的输出文件
```

## Markdown 语法

### 分页符

```markdown
{{< pagebreak >}}
```

### 代码块

带语法高亮的标准围栏代码块：

````markdown
```bash
npm install
```
````

### 表格

```markdown
| 列 A | 列 B |
|----------|----------|
| 值 1  | 值 2  |
```

### 图片

```markdown
![标题](path/to/image.png){width=50%}
```

## 自定义模板

### 扩展配置

创建 `_extensions/mytemplate/_extension.yml`：

```yaml
title: My Template
author: Your Name
version: 1.0.0
contributes:
  formats:
    typst:
      template: typst-template.typ
      template-partials:
        - typst-show.typ
```

### 设计系统（Typst）

```typst
// 配色方案（Slate + Indigo 调色板）
#let primary = rgb("#0f172a")      // Slate 900 - 标题
#let secondary = rgb("#334155")    // Slate 700 - 副标题
#let accent = rgb("#6366f1")       // Indigo 500 - 强调色
#let muted = rgb("#64748b")        // Slate 500 - 元数据
#let light-bg = rgb("#f8fafc")     // Slate 50 - 代码背景
#let border-light = rgb("#e2e8f0") // Slate 200 - 边框
```

### 排版

```typst
#set text(
  font: ("Inter", "Helvetica Neue", "Arial"),
  size: 11pt,
)

#set par(
  leading: 0.75em,
  justify: true,
)

// 代码块
#show raw.where(block: true): it => {
  block(
    fill: light-bg,
    stroke: (left: 3pt + accent),
    inset: 10pt,
    radius: 4pt,
    it,
  )
}
```

### 提示框

```typst
#let info(title: "Note", body) = {
  block(
    fill: rgb("#E0F2FE"),
    stroke: (left: 3pt + rgb("#0284C7")),
    inset: 12pt,
    [*#title*: #body]
  )
}

#let warning(title: "Warning", body) = {
  block(
    fill: rgb("#FEF3C7"),
    stroke: (left: 3pt + rgb("#D97706")),
    inset: 12pt,
    [*#title*: #body]
  )
}

#let success(title: "Success", body) = {
  block(
    fill: rgb("#DCFCE7"),
    stroke: (left: 3pt + rgb("#16A34A")),
    inset: 12pt,
    [*#title*: #body]
  )
}

#let danger(title: "Danger", body) = {
  block(
    fill: rgb("#FEE2E2"),
    stroke: (left: 3pt + rgb("#DC2626")),
    inset: 12pt,
    [*#title*: #body]
  )
}
```

## 故障排查

### 快速校验

```bash
# 检查 Quarto 版本
quarto --version  # >= 1.4.0

# 验证扩展是否存在
ls _extensions/*/

# 校验代码块是否成对（必须为偶数）
grep -c '^```' document.qmd

# 检查编码
file -i document.qmd  # 必须显示 utf-8
```

### 常见问题

| 问题 | 原因 | 解决方法 |
|-------|-------|-----|
| 嵌套代码块渲染异常 | 内层 ` ``` ` 提前关闭外层 | 外层使用 4 个及以上反引号 |
| 表格渲染为代码 | 上方 ` ``` ` 不匹配 | 检查分隔符数量 |
| 找不到扩展 | 目录路径错误 | 确认 `_extensions/` 路径 |
| 字体警告 | 字体未安装 | 正常现象，将使用回退字体 |
| 字符显示异常 | 编码错误 | 转换为 UTF-8 |

### 嵌套代码块

外层代码块使用更多反引号：

`````markdown
````markdown
# 这是外层代码块

```bash
echo "这是嵌套内容"
```

外层继续...
````
`````

### 校验脚本

```bash
#!/bin/bash
for f in *.qmd; do
  count=$(grep -c '^```' "$f")
  if [ $((count % 2)) -ne 0 ]; then
    echo "ERROR: $f has odd count ($count)"
  fi
done
```

### 完整校验流水线

```bash
#!/bin/bash
# validate-qmd.sh

echo "=== Validating QMD files ==="
errors=0

for f in *.qmd; do
  # 检查代码块是否成对
  count=$(grep -c '^```' "$f")
  if [ $((count % 2)) -ne 0 ]; then
    echo "ERROR: $f - odd code block count ($count)"
    ((errors++))
  fi

  # 检查 UTF-8 编码
  encoding=$(file -i "$f" | grep -o 'charset=[^;]*')
  if [[ "$encoding" != *"utf-8"* ]]; then
    echo "WARNING: $f - encoding is $encoding"
  fi
done

echo "=== Validation complete: $errors errors ==="
exit $errors
```

## 示例用例

### 技术文档

```yaml
---
title: "API 参考手册"
subtitle: "v2.0"
author: "工程团队"
date: 2026-01-17
format:
  typst:
    toc: true
    toc-depth: 3
---

# 身份认证

所有请求均需提供 API 密钥...
```

### 白皮书系列

```yaml
---
title: "安全最佳实践"
series: "工程白皮书"
wp-number: "03"
author: "安全团队"
date: 2026-01-17
format:
  whitepaper-typst:
    toc: true
---
```

### 内部报告

```yaml
---
title: "Q1 绩效报告"
author: "数据分析团队"
date: 2026-01-17
date-format: "Q1 YYYY"
format:
  typst:
    toc: false
---
```

## 参考资源

- [Quarto 文档](https://quarto.org/docs/guide/)
- [Typst 文档](https://typst.app/docs/)
- [Quarto + Typst 指南](https://quarto.org/docs/output-formats/typst.html)
- [工作流指南](../../guide/workflows/pdf-generation.md)
