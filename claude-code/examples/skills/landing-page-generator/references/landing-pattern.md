> 📚 **AI Spark Wiki** · Claude Code 知识库

# 落地页模式参考

记录了在 `claude-code-ultimate-guide-landing` 和 `claude-cowork-guide-landing` 中使用的既有落地页模式。

## 技术栈

| 组件 | 选择 | 理由 |
|-----------|--------|-----------|
| 框架 | 无（原生） | 简洁，无需构建步骤，易于托管 |
| 样式 | 单一 CSS 文件 | 可维护，无需预处理器 |
| JavaScript | 原生 + MiniSearch CDN | 依赖最少，懒加载 |
| 部署 | GitHub Pages + Actions | 免费、自动、可靠 |
| 搜索 | MiniSearch + 降级方案 | 客户端，速度快，无需后端 |

## 文件结构

```
project-landing/
├── index.html              # 主落地页（所有区块）
├── styles.css              # 完整样式表（约 3000 行）
├── search.js               # 搜索弹窗 + 键盘导航
├── search-data.js          # 搜索索引数组
├── *-data.js               # 额外数据文件（可选）
├── favicon.svg             # 项目图标
├── robots.txt              # SEO
├── CLAUDE.md               # Claude 指令
├── README.md               # 仓库文档
├── assets/                 # 图片、截图
└── .github/workflows/
    └── static.yml          # Pages 部署
```

## HTML 结构

### 文档头部

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Project Name] - [Tagline]</title>
  <meta name="description" content="[Description]">

  <!-- SEO -->
  <link rel="canonical" href="https://[user].github.io/[repo]-landing/">
  <meta name="robots" content="index, follow">

  <!-- Open Graph -->
  <meta property="og:type" content="website">
  <meta property="og:title" content="[Title]">
  <meta property="og:description" content="[Description]">
  <meta property="og:url" content="[URL]">
  <meta property="og:image" content="[og-image.png]">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="[Title]">
  <meta name="twitter:description" content="[Description]">

  <!-- Favicon -->
  <link rel="icon" type="image/svg+xml" href="favicon.svg">

  <!-- Styles -->
  <link rel="stylesheet" href="styles.css">
</head>
```

### Body 结构

```html
<body>
  <a href="#main" class="skip-link">Skip to content</a>

  <header class="header">...</header>

  <main id="main">
    <section class="hero">...</section>
    <section class="features">...</section>
    <section class="install">...</section>
    <section class="faq">...</section>
    <!-- 更多区块 -->
  </main>

  <footer class="footer">...</footer>

  <!-- 搜索弹窗 -->
  <div id="search-modal" class="search-modal" role="dialog" aria-modal="true">...</div>

  <!-- Scripts（顺序重要） -->
  <script src="search-data.js"></script>
  <script src="search.js"></script>
</body>
```

## 区块模式

### 头部导航

```html
<header class="header">
  <div class="container header-content">
    <a href="/" class="logo">
      <span class="logo-icon">>_</span>
      <span class="logo-text">[Project Name]</span>
    </a>
    <nav class="nav" aria-label="Main navigation">
      <ul class="nav-list">
        <li><a href="#features">Features</a></li>
        <li><a href="#install">Install</a></li>
        <li><a href="#faq">FAQ</a></li>
      </ul>
    </nav>
    <div class="header-actions">
      <button class="search-btn" aria-label="Search (Cmd+K)">
        <span>Search</span>
        <kbd>⌘K</kbd>
      </button>
      <a href="[github-url]" class="btn btn-github-star">
        ⭐ Star on GitHub
      </a>
    </div>
  </div>
</header>
```

### Hero 区块

```html
<section class="hero">
  <div class="container">
    <div class="hero-badges">
      <img src="https://img.shields.io/badge/..." alt="...">
      <!-- 更多徽章 -->
    </div>
    <h1 class="hero-title">[Main Title]</h1>
    <p class="hero-tagline">[Tagline/TL;DR]</p>
    <div class="hero-stats">
      <span class="stat"><strong>[N]</strong> features</span>
      <span class="stat"><strong>[N]</strong> examples</span>
    </div>
    <div class="hero-ctas">
      <a href="#install" class="btn btn-primary">Quick Start</a>
      <a href="[github]" class="btn btn-secondary">View on GitHub</a>
    </div>
  </div>
</section>
```

### 风险提示横幅（可选）

```html
<div class="risk-banner" role="alert">
  <div class="container">
    <span class="risk-icon">⚠️</span>
    <span class="risk-text">
      <strong>Risk Disclosure:</strong> [Warning text]
    </span>
    <a href="#risk-disclosure" class="risk-link">Learn more →</a>
  </div>
</div>
```

### 功能特性网格

```html
<section id="features" class="features">
  <div class="container">
    <h2 class="section-title">Features</h2>
    <div class="features-grid">
      <div class="feature-card">
        <div class="feature-icon">[emoji/icon]</div>
        <h3 class="feature-title">[Title]</h3>
        <p class="feature-desc">[Description]</p>
      </div>
      <!-- 更多卡片 -->
    </div>
  </div>
</section>
```

### 带复制功能的代码块

```html
<div class="code-block">
  <div class="code-header">
    <span class="code-lang">[language]</span>
    <button class="copy-btn" onclick="copyCode(this)" aria-label="Copy code">
      📋 Copy
    </button>
  </div>
  <pre><code>[code content]</code></pre>
</div>
```

### FAQ 区块

```html
<section id="faq" class="faq">
  <div class="container">
    <h2 class="section-title">FAQ</h2>
    <div class="faq-list">
      <details class="faq-item">
        <summary class="faq-question">[Question]?</summary>
        <div class="faq-answer">
          <p>[Answer]</p>
        </div>
      </details>
      <!-- 更多条目 -->
    </div>
  </div>
</section>
```

### 页脚

```html
<footer class="footer">
  <div class="container">
    <div class="footer-content">
      <div class="footer-brand">
        <span class="logo">>_ [Project]</span>
        <p class="footer-tagline">[Short tagline]</p>
      </div>
      <nav class="footer-links">
        <a href="[github]">GitHub</a>
        <a href="#faq">FAQ</a>
        <a href="[docs]">Docs</a>
      </nav>
      <div class="footer-meta">
        <span>MIT License</span>
        <span>v[version]</span>
      </div>
    </div>
  </div>
</footer>
```

## CSS 架构

### 自定义属性（主题）

```css
:root {
  /* 颜色 - 深色主题 */
  --color-bg: #0d1117;
  --color-surface: #161b22;
  --color-surface-hover: #21262d;
  --color-border: #30363d;
  --color-text: #c9d1d9;
  --color-text-muted: #8b949e;
  --color-heading: #f0f6fc;
  --color-primary: #58a6ff;
  --color-primary-hover: #79b8ff;
  --color-success: #3fb950;
  --color-warning: #d29922;
  --color-danger: #f85149;

  /* 间距 */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --space-2xl: 3rem;
  --space-3xl: 4rem;

  /* 排版 */
  --font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-mono: 'SF Mono', Consolas, 'Liberation Mono', monospace;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 2rem;
  --font-size-4xl: 2.5rem;

  /* 布局 */
  --container-max: 1200px;
  --radius: 6px;
  --radius-lg: 12px;

  /* 阴影 */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.3);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.3);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.3);
}
```

### 组件模式

```css
/* 容器 */
.container {
  max-width: var(--container-max);
  margin: 0 auto;
  padding: 0 var(--space-lg);
}

/* 按钮 */
.btn {
  display: inline-flex;
  align-items: center;
  gap: var(--space-sm);
  padding: var(--space-sm) var(--space-lg);
  border-radius: var(--radius);
  font-weight: 500;
  text-decoration: none;
  transition: all 0.2s;
}

.btn-primary {
  background: var(--color-primary);
  color: var(--color-bg);
}

.btn-secondary {
  background: var(--color-surface);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}

/* 卡片 */
.feature-card {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  padding: var(--space-xl);
  transition: transform 0.2s, box-shadow 0.2s;
}

.feature-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

/* 网格 */
.features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-xl);
}
```

### 响应式断点

```css
/* 平板 */
@media (max-width: 768px) {
  .hero-title { font-size: var(--font-size-3xl); }
  .header-actions { display: none; }
  .nav { display: none; }
  /* 移动端导航切换 */
}

/* 手机 */
@media (max-width: 480px) {
  .hero-ctas { flex-direction: column; }
  .features-grid { grid-template-columns: 1fr; }
}
```

## JavaScript 模式

### 搜索实现

```javascript
(function() {
  'use strict';

  let searchIndex = null;
  let miniSearchLoaded = false;

  // 懒加载 MiniSearch
  async function loadMiniSearch() {
    if (miniSearchLoaded) return;
    await loadScript('https://cdn.jsdelivr.net/npm/minisearch@7/dist/umd/index.min.js');
    miniSearchLoaded = true;
  }

  // 从 window.SEARCH_* 数据构建索引
  function buildIndex() {
    const items = [
      ...(window.SEARCH_FEATURES || []),
      ...(window.SEARCH_FAQ || []),
    ];
    // ... 索引构建
  }

  // 键盘导航
  document.addEventListener('keydown', (e) => {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
      e.preventDefault();
      openSearchModal();
    }
  });
})();
```

### 复制代码函数

```javascript
async function copyCode(button) {
  const codeBlock = button.closest('.code-block');
  const code = codeBlock.querySelector('code').textContent;

  try {
    await navigator.clipboard.writeText(code);
    button.textContent = '✓ Copied!';
    setTimeout(() => {
      button.textContent = '📋 Copy';
    }, 2000);
  } catch (err) {
    console.error('Copy failed:', err);
  }
}
```

## 部署

### GitHub Actions 工作流

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

## 无障碍访问检查清单

- [ ] 跳转到主内容的跳过链接
- [ ] 语义化 HTML（header、main、section、footer）
- [ ] 交互元素的 ARIA 标签
- [ ] 弹窗的键盘导航
- [ ] 焦点可见样式
- [ ] 颜色对比度符合 WCAG AA
- [ ] 尊重减少动效设置
- [ ] 图片的 alt 文本

## SEO 检查清单

- [ ] 描述性 title 标签
- [ ] Meta description
- [ ] 规范 URL（Canonical URL）
- [ ] Open Graph 标签
- [ ] Twitter Card 标签
- [ ] 结构化数据（Schema.org）
- [ ] robots.txt
- [ ] 语义化标题层级
