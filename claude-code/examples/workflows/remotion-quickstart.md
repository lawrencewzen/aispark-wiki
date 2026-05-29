> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Remotion + Claude Code 快速入门"
description: "15 分钟快速入门：使用 Remotion 和 Claude Code 创建程序化视频"
tags: [tutorial, workflow, integration]
---

# Remotion + Claude Code 快速入门

**目标**：在 15 分钟内使用 Remotion 和 Claude Code 创建你的第一个程序化视频。

**难度**：React 入门级（掌握基础知识即可）

---

## 前置要求

| 必需项 | 最低版本 | 验证命令 |
|--------|---------|---------|
| **Node.js** | 18+ | `node --version` |
| **npm** | 8+ | `npm --version` |
| **Claude Code** | 2.1+ | `claude --version` |
| **React 基础** | JSX 语法 | 了解 `<div>`、`props`、`useState` |

**预计耗时**：15-20 分钟（首次操作）

---

## 第一步：安装 Remotion 技能包

### 方案 A：通过 skills.sh 安装（推荐）

```bash
# 在新项目目录下执行
mkdir my-remotion-test && cd my-remotion-test

# 为 Claude Code 安装 Remotion 技能包
npx skills add remotion-dev/skills
```

**预期输出**：
```
✓ Skills added to .claude/skills/
  - remotion-best-practices
  - remotion-animations
  - remotion-audio
  [... 已安装 20+ 个技能]
```

### 方案 B：手动安装（skills.sh 不可用时）

```bash
# 直接克隆技能包仓库
git clone https://github.com/remotion-dev/skills.git .claude/skills/remotion
```

---

## 第二步：创建第一个 Remotion 项目

### 通过 Claude Code（推荐方式）

启动 Claude Code 并使用以下提示词：

```
Create a new Remotion project for a simple 5-second video with:
- A fade-in title "Hello Remotion"
- Background gradient (blue to purple)
- Smooth animation

Use npx create-video to scaffold the project.
```

**Claude 将会执行**：
1. 运行 `npx create-video@latest my-video`
2. 生成 Remotion 样板代码
3. 为你的视频创建 React 组件
4. 配置 `remotion.config.ts`

### 预期目录结构

```
my-remotion-test/
├── src/
│   ├── Root.tsx           # 入口文件
│   ├── HelloWorld.tsx     # 你的视频合成组件
│   └── ...
├── public/
├── package.json
└── remotion.config.ts
```

---

## 第三步：预览你的视频

### 启动 Remotion Studio

```bash
npm start
```

**结果**：本地服务器在 `http://localhost:3000` 启动

**Studio 界面功能**：
- 交互式时间轴（可拖动查看每一帧）
- 合成组件的实时预览
- 播放控制
- 属性面板（实时修改参数）

### 测试修改

在 `src/HelloWorld.tsx` 中修改文本：

```tsx
<h1 style={{fontSize: 100}}>Hello Remotion!</h1>
```

**热重载** → 修改立即在 Studio 中生效。

---

## 第四步：渲染你的视频

### 基础渲染命令

```bash
npm run build
```

**执行过程**：
1. Remotion 编译你的 React 代码
2. 生成每一帧（30 fps × 时长 = 5 秒对应 150 帧）
3. FFmpeg 将帧序列编码为 MP4
4. 输出文件：`out/video.mp4`

### 高级渲染选项

```bash
# 高画质（1080p，60fps）
npx remotion render src/index.ts HelloWorld out/video.mp4 \
  --width 1920 \
  --height 1080 \
  --fps 60

# GIF 格式
npx remotion render src/index.ts HelloWorld out/video.gif \
  --image-format png
```

---

## 第五步：配合 Claude Code 迭代优化

### 高效提示词示例

**示例 1：添加动画**
```
Add a smooth scale animation to the title:
- Start at scale 0.8
- End at scale 1.0
- Use spring physics with config {damping: 20}
```

**示例 2：添加音频**
```
Add background music to the video:
- Use a public domain track from /public/music.mp3
- Fade in over 1 second
- Volume at 0.5
```

**示例 3：多场景序列**
```
Create a 3-scene video:
1. Intro (0-2s): Logo fade in
2. Main (2-8s): Product showcase with transitions
3. Outro (8-10s): Call to action text
```

### 高效 Claude Code 工作流模式

```
1. 用自然语言描述需求
   └─ Claude 生成 JSX + 动画代码

2. 在 Studio 中预览
   └─ 目视验证效果

3. 根据具体反馈迭代
   └─ "Make the transition slower"
   └─ "Change color to #FF6B35"

4. 渲染最终视频
   └─ npm run build
```

---

## 故障排查

### 问题："Command not found: remotion"

**解决方案**：
```bash
# 全局安装
npm install -g @remotion/cli

# 或使用 npx
npx remotion --version
```

### 问题："React is not defined"

**解决方案**：检查 `package.json` 中是否包含 `react` 和 `react-dom`：
```bash
npm install react react-dom
```

### 问题：渲染时出现 FFmpeg 错误

**解决方案**：安装 FFmpeg：
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
choco install ffmpeg
```

### 问题：Claude 生成的代码无法编译

**原因**：技能包未正确加载。

**解决方案**：
```bash
# 检查 .claude/skills/ 是否存在
ls -la .claude/skills/remotion

# 重新加载技能包
claude --reload-skills
```

### 问题：视频已渲染但画面全黑

**调试步骤**：
1. 检查 Studio 控制台（F12）
2. 查找 TypeScript 报错
3. 验证 `<Composition>` 的属性是否正确：
   ```tsx
   <Composition
     id="HelloWorld"
     component={HelloWorld}
     durationInFrames={150}  // 5s × 30fps
     fps={30}
     width={1920}
     height={1080}
   />
   ```

---

## 成功指标

完成本快速入门后，你应该已经：

- ✅ 搭建好可运行的 Remotion 项目（已生成脚手架）
- ✅ 渲染出测试视频（`out/` 目录下的 MP4 文件）
- ✅ 理解 Claude → 预览 → 渲染的工作流
- ✅ 通过 Claude 提示词创建了 1-2 个动画

**典型渲染耗时**（5 秒视频，1080p）：
- MacBook Pro M1：约 10-15 秒
- Intel i5：约 30-45 秒
- 云端（GitHub Actions）：约 60-90 秒

---

## 后续步骤

### 中级进阶

1. **添加素材资源**：
   ```
   Claude, import an image from /public/logo.png and animate it rotating 360°
   ```

2. **数据驱动视频**：
   ```tsx
   // 从数据集批量生成 100 个视频
   const data = [{name: "John", score: 95}, ...];
   npx remotion render --props='{"data": data}'
   ```

3. **复杂合成组件**：
   ```
   Create a video with 3 scenes using <Sequence>:
   - Scene 1: Intro (0-30 frames)
   - Scene 2: Content (30-150 frames)
   - Scene 3: Outro (150-180 frames)
   ```

### 参考资源

| 资源 | 链接 | 类型 |
|------|------|------|
| **Remotion 文档** | [remotion.dev/docs](https://www.remotion.dev/docs/) | 官方文档 |
| **Agent 技能仓库** | [github.com/remotion-dev/skills](https://github.com/remotion-dev/skills) | GitHub |
| **Discord 社区** | [remotion.dev/discord](https://www.remotion.dev/discord) | 社区支持（约 1.5K 成员） |
| **示例展示** | [remotion.dev/showcase](https://www.remotion.dev/showcase) | 灵感参考 |

---

## 已验证的应用场景

| 使用场景 | 示例 | 难度 |
|---------|------|------|
| **产品演示** | 带高亮的功能演示 | ⭐⭐ |
| **YouTube 片头** | 动画 Logo + 标题卡 | ⭐ |
| **数据可视化** | 动画图表、信息图 | ⭐⭐⭐ |
| **社交媒体** | Instagram Story、TikTok 模板 | ⭐⭐ |
| **说明视频** | 步骤式动画教程 | ⭐⭐⭐⭐ |

**成功案例**：Icon.me（年收入 $500 万）、Submagic（年收入 $800 万）、Crayo（月收入 $50 万）

---

## 重要局限性

1. **需要 React 知识**：Claude 可以协助，但理解 JSX 是调试的基本前提
2. **有一定学习曲线**：制作前几个视频需要 2-4 小时；熟练掌握需要 2-4 周
3. **费用问题**：商业许可证（超过 3 人使用）+ Claude API 费用 + 渲染计算费用
4. **并非 After Effects 的替代品**：编程方式与时间轴方式是不同的创作范式

---

## 完整会话示例

```bash
# 1. 初始化设置
mkdir remotion-demo && cd remotion-demo
npx skills add remotion-dev/skills

# 2. 启动 Claude Code
claude

> Create a 10-second countdown timer video:
> - Large numbers (1-10) centered
> - Each number appears for 1 second
> - Use spring animations for each transition
> - Background gradient that changes color per number
> - Add a "beep" sound effect on each number change

# Claude 生成代码...

# 3. 预览
npm start
# 打开 http://localhost:3000

# 4. 调整优化
> Make the spring animation bouncier (increase damping to 40)
> Change gradient colors to warm tones (orange to red)

# 5. 最终渲染
npm run build

# 结果：out/video.mp4（10 秒，300 帧）
```

**整体会话耗时**：约 25 分钟（包含迭代调整）

---

## 上线前检查清单

- [ ] 测试完整渲染（不仅是预览）
- [ ] 检查性能（watch 模式与生产模式的差异）
- [ ] 验证素材资源（许可证、分辨率、格式）
- [ ] 规划渲染费用（云端渲染 vs 本地渲染）
- [ ] 记录动态属性文档（用于数据驱动场景）
- [ ] 如需自动生成，配置 CI/CD（GitHub Actions + Remotion Lambda）

---

## 补充学习资源

### 必读精选（从这里开始）

| 资源 | 类型 | 链接 | 推荐理由 |
|------|------|------|---------|
| **官方资源中心** | 资源汇总 | [remotion.dev/docs/resources](https://www.remotion.dev/docs/resources) | 50+ 模板、集成方案、特效 |
| **Fireship 教程** | 视频（8 分钟） | [This video was made with code](https://www.youtube.com/watch?v=deg8bOoziaE) | 超快速入门，100 万+ 播放量 |
| **Discord 社区** | 实时支持 | [Remotion Discord](https://discord.com/servers/remotion-809501355504959528) | 5,600+ 成员，内置 AI 机器人 |

### 推荐文字教程

| 文章 | 难度 | 链接 | 特色 |
|------|------|------|------|
| **ClipCat 入门指南** | 入门 | [Create Videos Programmatically](https://www.clipcat.com/blog/create-videos-programmatically-using-react-a-beginners-guide-to-remotion/) | 分步安装说明 |
| **Prismic 教程** | 入门 | [Learn to Create Videos](https://prismic.io/blog/create-videos-with-code-remotion-tutorial) | 完整基础讲解 |
| **SitePoint 介绍** | 中级 | [Remotion Tutorial](https://www.sitepoint.com/remotion-create-animated-videos-using-html-css-react/) | 数据获取、组件开发 |

### 按目标选择的视频教程

**快速上手**（15 分钟）：
- [Fireship - This video was made with code](https://www.youtube.com/watch?v=deg8bOoziaE)（8:41，100 万播放量）

**深度学习**（1-2 小时）：
- [CoderOne - Create Videos with React](https://www.youtube.com/watch?v=VOX98RoITMk)（1 小时，完整动画 Logo 示例）

**Claude Code 集成**（30 分钟）：
- [Snapper AI - Generate Animated Videos](https://www.youtube.com/watch?v=EwKCAgt4aKI)（9:48，2026 年 1 月）
- [chantastic - Making Remotion Videos](https://www.youtube.com/watch?v=z87bczUZ0uo)（30 分钟，2026 年 1 月）

### 模板与示例

| 模板 | 适用场景 | 链接 | 复杂度 |
|------|---------|------|--------|
| **Hello World** | 第一个项目 | [官方模板](https://www.remotion.dev/templates/) | ⭐ 简单 |
| **Audiogram** | 播客可视化 | [官方模板](https://www.remotion.dev/templates/) | ⭐⭐ 中等 |
| **GitHub Unwrapped** | 真实生产案例 | [github.com/remotion-dev/github-unwrapped](https://github.com/remotion-dev/github-unwrapped) | ⭐⭐⭐ 进阶 |

**完整列表**：[remotion.dev/docs/resources](https://www.remotion.dev/docs/resources)（50+ 持续维护的模板）

### 成功案例（激励参考）

使用 Remotion 实现盈利的真实产品：

- **Icon.me**：30 天内年化收入达 $500 万（广告素材生成工具）
- **Revid.ai**：15 个月内年化收入达 $100 万（AI 视频平台）
- **Typeframes**：已被收购（产品演示视频工具）

[查看全部展示案例](https://www.remotion.dev/showcase)

### 提问渠道

| 平台 | 链接 | 最适合 | 活跃度 |
|------|------|--------|--------|
| **Discord** | [Remotion Discord](https://discord.com/servers/remotion-809501355504959528) | 快速支持，AI 机器人 | ⭐⭐⭐⭐⭐ 非常活跃 |
| **GitHub Discussions** | [remotion-dev/discussions](https://github.com/orgs/remotion-dev/discussions) | 架构问题 | ⭐⭐⭐ 中等活跃 |
| **Stack Overflow** | [#remotion 标签](https://stackoverflow.com/questions/tagged/remotion) | 具体问题 | ⭐⭐ 低活跃 |

**小技巧**：在 Discord 上 @CrawlChat AI 机器人，可获得带文档来源的即时回答。

### 进阶资源

**性能与优化**：
- [官方性能文档](https://www.remotion.dev/docs/performance)
- [YouTube：优化 Remotion Lambda](https://www.youtube.com/results?search_query=optimizing+remotion+lambda+jonny+burger)（Jonny Burger，17 分钟）

**Agent 技能与 AI**：
- [官方 AI 技能文档](https://www.remotion.dev/docs/ai/skills)
- [AIbase 文章](https://news.aibase.com/news/24827)（2026 年 1 月，最新资讯）

**部署**：
- [Railway 模板](https://railway.com/deploy/remotion-on-rails)（一键部署）
- [Lambda 文档](https://www.remotion.dev/docs/lambda)（AWS 无服务器渲染）

### 实用扩展包

| 包名 | 用途 | 安装命令 |
|------|------|---------|
| `@remotion/shapes` | SVG 形状（三角形、星形等） | `npm i @remotion/shapes` |
| `remotion-transition-series` | 场景间过渡效果 | `npm i remotion-transition-series` |
| `remotion-subtitle` | 自动字幕 | `npm i remotion-subtitle` |
| `@remotion/tailwind` | 集成 Tailwind CSS | `npm i @remotion/tailwind` |

[完整列表见资源页](https://www.remotion.dev/docs/resources)

---

## 建议学习路径

### 第一周：基础入门（6-8 小时）
1. 阅读官方文档（2 小时）
2. 观看 Fireship 教程（15 分钟）
3. 跟随 ClipCat 指南完成 Hello World（2 小时）
4. 复现 CoderOne 示例（2-3 小时）
5. 加入 Discord，积极提问

### 第二周：动手实践（8-10 小时）
1. 制作 3 个不同类型的视频（片头、倒计时、数据可视化）
2. 研究 GitHub Unwrapped 源码
3. 测试 Claude Code 集成（Agent 技能）
4. 探索扩展包（`@remotion/shapes`、过渡效果）

### 第三周：投入生产（持续进行）
1. 针对具体使用场景创建真实项目
2. 优化性能（如有需要则使用 Lambda）
3. 配置 CI/CD（GitHub Actions）
4. 在 Discord 分享作品，收集反馈

---

**创建时间**：2026-01-23
**最后更新**：2026-01-24（补充资源）
**测试环境**：Claude Code 2.1.17、Remotion 4.x、Node.js 20.x
**平均耗时**：15-20 分钟（第一个视频），5-10 分钟（后续视频）
**资源验证时间**：2026-01-24（Perplexity Pro，50+ 来源）
