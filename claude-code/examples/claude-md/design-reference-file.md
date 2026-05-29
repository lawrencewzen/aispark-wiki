> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "设计参考文件"
description: "将 brand-book.html 和 ui-kit.html 保存在项目根目录，作为 Claude Code 的永久上下文，以实现一致的 UI 生成"
tags: [design-system, frontend, web, ui, consistency, brand, color-palette, tailwind]
---

# 设计参考文件 — CLAUDE.md 模式

将 `brand-book.html` 和 `ui-kit.html` 保存在项目根目录，作为永久上下文文件。Claude Code 在生成任何 UI 之前都会读取这两个文件——每个新页面都会自动继承你的设计系统。

灵感来源：Boris Paillard 的工作流（mixt.care，2026 年 3 月）：设计系统一旦建立，新页面只需 5 分钟而非 30 分钟。

## 问题

在多个会话中使用 Claude Code 构建网站时，每个新页面都面临偏离设计的风险——颜色错误、字体不一致、出现新的组件变体。在每个提示词中重复说明设计约束既繁琐又不可靠。

## 解决方案

在项目根目录放置两个 HTML 文件，作为 Claude 随时可读取的设计记忆：

- `brand-book.html` — 带语义角色的调色板、字体、CSS 变量、WCAG 对比度
- `ui-kit.html` — 已记录的组件库（按钮、表单、信任栏、区块标签）

在 CLAUDE.md 中加入一条指令，让 Claude 在每次 UI 任务前都引用这两个文件。

## 项目结构

```
project/
├── brand-book.html     # 调色板 + 字体 + CSS 变量（永久参考）
├── ui-kit.html         # 使用 Tailwind 记录的组件库
├── CLAUDE.md           # 下方的设计系统指令
├── src/
│   ├── styles/
│   │   └── tokens.css  # 从 brand-book.html 提取的 CSS 变量
│   └── components/
└── ...
```

## CLAUDE.md 片段

将以下内容添加到项目级 `CLAUDE.md`：

```markdown
## 设计系统

在生成任何 UI 组件、页面或布局之前，务必先读取 `brand-book.html` 和 `ui-kit.html`。

规则：
- 禁止硬编码颜色或字体大小——使用 brand-book.html 中的 CSS 变量
- 优先复用 ui-kit.html 中的组件，不要创建新变体
- 新页面必须使用相同的设计 token（--color-primary、--font-primary 等）
- 若请求的设计元素不在 UI 套件中，创建后需在 ui-kit.html 中记录
```

## 第一步 — 生成 brand-book.html

```
在项目根目录创建 brand-book.html。

对调色板中的每种颜色，展示一张卡片，包含：
- 色块（120×80px）
- 名称、十六进制色值、RGB 值、HSL 值
- CSS 变量名（例如 --color-primary）
- 语义角色：PRIMARY | DARK | LIGHT | ACCENT | NEUTRAL
- 纯文字使用规则（例如"CTA、按钮、激活链接"）
- 与白色的 WCAG 对比度：X.X:1 — AA PASS/FAIL — AAA PASS/FAIL
- 与黑色的 WCAG 对比度：X.X:1 — AA PASS/FAIL — AAA PASS/FAIL

我的调色板：
[在此列出你的颜色和角色]

我的字体：
[在此列出你的字体]

包含字型比例区块：12px / 16px / 20px / 24px / 32px / 48px / 64px / 96px，附 rem 等值。

在底部输出一个可复制的 <style> 块，包含所有 CSS 变量（:root { ... }）。

brand-book.html 本身使用这些变量来设计样式——它应当展示这套设计系统。
```

## 第二步 — 生成 ui-kit.html

```
使用 Tailwind 和 brand-book.html 中的 CSS 变量构建 ui-kit.html，记录基础组件。

包含：
- 按钮：主要/次要/幽灵变体，3 种尺寸（sm/md/lg）
- 带垂直分隔线的信任栏
- 带水平分割线的区块标签（大写，字间距）
- 字体排版示例：h1–h4、正文、说明文字——使用 --font-primary 和 --font-secondary
- 带对比度展示的调色板网格
- 表单元素：input、select、textarea 的默认/聚焦/错误状态

每个组件应展示：组件名称、使用说明及所用的 Tailwind 类。
引用 brand-book.html 中的 CSS 变量——禁止硬编码值。
```

## 第三步 — WCAG 色彩无障碍审查（可选，但推荐）

在生成品牌手册后运行，提前发现可访问性问题：

```
审查 brand-book.html 中调色板的 WCAG 2.1 合规性：

1. 计算所有可能的前景/背景颜色组合的精确对比度（保留 2 位小数）
2. 标记不符合 WCAG AA 的配对：正文文本 4.5:1、大文本及 UI 组件 3:1
3. 模拟红绿色盲、红色盲、蓝黄色盲——标记有问题的配对
4. 对每对不合格配对，建议在色相偏差不超过 10% 的情况下通过 AA 所需的最小十六进制调整
5. 输出：Markdown 表格，列名：配对组合 | 对比度 | AA 正文 | AA 大文本 | AAA | 色盲安全

外部参考：
- WebAIM 对比度检查器：https://webaim.org/resources/contrastchecker/
- ColorOracle（色盲模拟器）：https://colororacle.org/
```

## 第四步 — 滚动动画（可选）

```
使用原生 JS Intersection Observer 为所有卡片和区块标题添加滚动动画。
元素在进入视口时应以 0.6s ease-out 过渡淡入并向上滑动，相邻元素错开 100ms。
仅对带 [data-animate] 属性的元素生效——按需启用，不全局应用。
禁止使用任何动画库——仅限原生 JS。
```

## 示例 — mixt.care 调色板

结构良好的调色板，含语义角色和 CSS 变量：

```css
:root {
  /* 颜色 */
  --color-primary: #5C1A2E;   /* 深波尔多红——CTA、按钮、激活链接 */
  --color-dark: #3D2B1F;      /* 暖棕——正文、深色背景 */
  --color-accent: #B87333;    /* 铜色——悬浮状态、次要高亮 */
  --color-secondary: #6B5B8E; /* 柔和紫——徽章、插图 */
  --color-light: #F5F0EA;     /* 奶油色——区块背景、卡片 */

  /* 字体 */
  --font-primary: 'Geist', sans-serif;       /* 正文 */
  --font-secondary: 'Newsreader', serif;     /* 强调、引用 */
  --font-display: 'Fraunces', serif;         /* Logo、展示性标题 */

  /* 字型比例 */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.25rem;
  --text-xl: 1.5rem;
  --text-2xl: 2rem;
  --text-3xl: 3rem;
  --text-display: 6rem;        /* 桌面端声明式 footer */
  --text-display-mobile: 3rem;
}
```

**该调色板的 WCAG 说明**（估算值）：
- 波尔多红配奶油色：约 8.2:1 — AA 通过
- 棕色配奶油色：约 9.1:1 — AA 通过
- 白色配波尔多红：约 8.2:1 — AA 通过
- 铜色配白色：约 3.1:1 — 正文 AA 不通过（仅用于 UI 组件，或加深至 #9B5F20）
- 紫色配奶油色：约 4.8:1 — AA 通过

## 适用场景

- 使用 Claude Code 构建营销网站、落地页或产品网站
- 需要在不同会话中生成多个页面时保持设计一致性
- 在已有品牌规范的项目中开始 UI 工作之前
- 用可交互、自我展示的参考文件替代手动编写的设计系统文档

## 局限性

- 适用于静态/营销网站和 MVP——复杂组件库需要专业设计系统工具（Storybook、Figma）
- 2 小时的时间估算前提是：开始提示前你已明确设计意图（颜色、字体、布局方向）
- Claude 的 WCAG 审查是估算值——关键配对请使用 WebAIM 或专用工具验证
