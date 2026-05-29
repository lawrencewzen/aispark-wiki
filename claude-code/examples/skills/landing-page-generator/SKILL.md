> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: landing-page-generator
description: "从任意代码库生成完整、可直接部署的落地页。适用于为开源项目创建主页、构建项目网站、将 README 转换为营销页面，或统一多个代码库的落地页风格。"
allowed-tools: Read Bash Write
effort: medium
---

# 落地页生成器

通过分析代码库的文档和结构，生成完整、可直接部署的落地页。

## 适用场景

- 为 GitHub 代码库创建落地页
- 从现有文档生成静态站点
- 统一多个项目的落地页风格
- 将 README 内容转换为营销/展示页面

## 技能功能说明

1. **分析代码库**：读取 README.md、CHANGELOG.md、package.json/VERSION、docs/、assets/
2. **提取内容**：识别标题、标语、功能特性、安装方式、截图
3. **映射章节**：主视觉区、功能区、安装区、FAQ、页脚（可选：风险提示横幅、定价表）
4. **生成落地页**：创建完整静态站点（HTML + CSS + JS）
5. **可部署输出**：包含用于 GitHub Pages 的 GitHub Actions 工作流

## 使用方式

### 基础用法

```
/landing-page-generator from ~/path/to/repo
```

### 带选项用法

```
/landing-page-generator from ~/path/to/repo --risk-banner --pricing-table
```

### 可用选项

| 选项 | 说明 | 默认值 |
|--------|-------------|---------|
| `--risk-banner` | 在首屏上方添加醒目的警告/免责声明横幅 | false |
| `--pricing-table` | 包含定价对比区域 | false |
| `--screenshots <path>` | 截图文件夹路径 | ./assets/ |
| `--theme [dark\|light]` | 配色主题 | dark |
| `--search` | 启用 Cmd+K 搜索 | true |
| `--output <path>` | 输出目录 | ./[repo-name]-landing/ |

## 工作流

### 第一步：代码库分析

读取并分析源代码库中的以下文件：

```
README.md        → 主要内容来源（标题、标语、功能、安装）
CHANGELOG.md     → 版本信息、近期变更
package.json     → 版本号、依赖、元数据
VERSION          → 备用版本来源
docs/            → 附加文档页面
assets/          → 截图、图片
LICENSE          → 徽章所需的许可证类型
```

### 第二步：内容提取映射

| 来源 | 目标章节 | 提取方式 |
|--------|---------------|-------------------|
| README 标题/徽章 | 主视觉区 | 第一个 H1 + shield.io 徽章行 |
| README TL;DR | 主视觉区标语 | 标题后的第一段或引用块 |
| README 功能特性 | 功能网格 | 带项目符号列表的 H2/H3 章节 |
| README 安装 | 快速开始 | 包含 shell 命令的代码块 |
| README 用法 | 示例 | 包含示例的代码块 |
| README FAQ | FAQ | Details/summary 或 H3+P 模式 |
| CHANGELOG | 最新动态 | 最近 1-3 个版本 |
| assets/*.png | 截图 | 画廊区域 |

### 第三步：章节生成

按顺序生成以下章节：

1. **顶部导航**（固定）
   - Logo/项目名称
   - 导航链接：功能、安装、FAQ
   - 操作按钮：搜索（Cmd+K）、GitHub Star、主要行动号召

2. **风险提示横幅**（若启用 `--risk-banner`）
   - 橙色/警告样式，位于首屏上方
   - 清晰可见的免责声明文字
   - 链接至详细披露章节

3. **主视觉区**
   - 来自 README H1 的标题
   - 来自 TL;DR/第一段的标语
   - 统计徽章（版本、许可证、平台）
   - 行动号召：「快速开始」（主要）、「在 GitHub 上查看」（次要）

4. **架构/概览**（若 README 中有架构图）
   - ASCII 图转换为样式化代码块
   - 或概览卡片

5. **功能网格**
   - 来自 README 功能特性的 4-6 个功能卡片
   - 图标 + 标题 + 描述的展示模式

6. **定价表**（若启用 `--pricing-table`）
   - 方案对比表
   - 如有，包含倍率/用量表

7. **截图画廊**（若 assets 存在）
   - 标签式或轮播式画廊
   - 来自 alt 文本的图片说明

8. **快速开始章节**
   - 一行安装命令（特色代码块）
   - 配置步骤
   - 第一个使用示例

9. **风险披露**（若启用 `--risk-banner`）
   - 完整免责声明章节
   - 服务条款注意事项
   - 建议说明

10. **FAQ 章节**
    - 从 README FAQ 或常见问题生成
    - 可折叠的 details 模式

11. **相关项目**（若 README 中有相关链接）
    - 链接至依赖/相关代码库的卡片

12. **页脚**
    - 快速链接
    - 许可证徽章
    - 版本信息
    - 作者/代码库链接

### 第四步：输出结构

```
[project-name]-landing/
├── index.html              # 主落地页
├── styles.css              # 完整样式表
├── search.js               # Cmd+K 搜索功能
├── search-data.js          # 搜索索引（FAQ、功能特性）
├── favicon.svg             # 生成或复制
├── robots.txt              # SEO
├── CLAUDE.md               # 项目说明
├── README.md               # 落地页代码库文档
├── assets/                 # 复制的截图
│   └── [copied from source]
└── .github/
    └── workflows/
        └── static.yml      # GitHub Pages 部署
```

### 第五步：验证检查点

最终确认前，请验证：
- 所有章节在浏览器中正常渲染
- 链接指向有效目标（GitHub 代码库、文档、安装命令）
- 响应式布局在移动端（375px）、平板（768px）和桌面端（1280px）宽度下正常显示
- 无障碍性：存在跳过链接、交互元素有 ARIA 标签、色彩对比度符合 WCAG AA

## 技术栈

- **无需构建步骤**：纯 HTML + CSS + JS
- **搜索**：从 CDN 懒加载 MiniSearch，带降级方案
- **部署**：通过 Actions 部署到 GitHub Pages
- **样式**：CSS 自定义属性，响应式，默认深色主题
- **无障碍性**：跳过链接、ARIA 标签、键盘导航

## CSS 模式（来自已有落地页）

### 组件类名

```css
/* 按钮 */
.btn, .btn-primary, .btn-secondary, .btn-github-star, .btn-outline

/* 卡片 */
.feature-card, .comparison-card, .path-card

/* 布局 */
.container, .features-grid, .hero, .section

/* 工具类 */
.visually-hidden, .skip-link
```

### CSS 变量

```css
:root {
  --color-bg: #0d1117;
  --color-surface: #161b22;
  --color-border: #30363d;
  --color-text: #c9d1d9;
  --color-text-muted: #8b949e;
  --color-primary: #58a6ff;
  --color-success: #3fb950;
  --color-warning: #d29922;
  --color-danger: #f85149;
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --radius: 6px;
}
```

## 示例

**用户**：`/landing-page-generator from ~/projects/my-project --risk-banner --pricing-table`

**输出**：

在 `~/projects/my-project-landing/` 下创建：
- 展示多提供商路由器的完整落地页
- 醒目的服务条款风险横幅（橙色，位于首屏上方）
- 提供商卡片（Anthropic、Copilot、Ollama）
- 来自 README 的定价表
- 截图画廊
- 已准备好 GitHub Pages 部署

## 使用建议

- 对于涉及法律/服务条款的项目，始终加上 `--risk-banner`
- 截图能显著提升落地页质量——确保 assets/ 目录已填充内容
- 本技能会保留 README 的语言（英语/法语）
- 请审查生成的 FAQ——可能需要自定义调整
- 生成后测试响应式设计

## 参考资料

详细模式文档见 `references/landing-pattern.md`。
可复用模板和代码片段见 `assets/`。
