> 📚 **AI Spark Wiki** · Claude Code 知识库

# Kimi 演示文稿模板

> 将整个提示词复制粘贴到 Kimi.com 以生成演示文稿。
> 发送前请填写所有 {占位符} 的值。
> 在文件中搜索 `{` 以找到剩余的占位符。

---

请按照以下规格创建一份专业的会议演讲演示文稿：

## 演示文稿要求

**标题**：{FULL_TITLE}
**演讲者**：{SPEAKER_NAME}
**活动**：{EVENT_NAME} — {MONTH_YEAR}
**目标受众**：{AUDIENCE_DESCRIPTION}
**时长**：{DURATION} 分钟（{SLIDE_COUNT} 张幻灯片）
**语言**：{LANGUAGE}
**格式**：会议演讲——叙事性、文字精简、大数字、高视觉冲击力

## 设计要求

**视觉风格**：
- 深色主题，现代简约
- 以最少文字实现最大视觉冲击
- 关键数字使用大字体（指标最小 48pt）
- 尽可能用图标代替项目符号
- 简洁无衬线字体（Inter、SF Pro 或类似字体）
- 即使在深色背景上也要留有充足的留白
- 高对比度，确保在投影仪屏幕上清晰可读

**配色方案**：
- 主背景：近黑色 (#0a0a0a)
- 表面背景：深灰 (#141414) — 用于卡片、内容块
- 浮层背景：(#1e1e1e) — 用于高亮区域
- 边框：(#2a2a2a) — 元素间的细微分隔
- 主文字：米白色 (#e5e5e5)
- 次要文字：中灰 (#a3a3a3)
- 静音文字：(#8a8a8a) — 用于标签、说明
- 强调色 - 橙色：(#f97316) — 用于关键数字、高亮、行动号召
- 强调悬停 - 浅橙：(#fb923c) — 用于次要高亮
- 成功绿：(#22c55e) — 仅用于正向指标
- 警告红：(#ef4444) — 仅用于问题、错误、负向指标

**布局偏好**：
- 每张幻灯片只有 1 个想法——绝不超过
- 一致的页脚，含幻灯片编号（低调，右下角）
- 左对齐文字，提高可读性
- 每张幻灯片最多 30 个字（不含标题）
- 当数字是主要内容时，以超大尺寸居中展示
- 代码块使用稍亮的背景 (#1e293b) 和等宽字体
- 图表使用简单方框、箭头和强调色配色

## 幻灯片内容结构

### 第一幕：{ACT1_TITLE}（幻灯片 1-{ACT1_LAST_SLIDE}，约 {ACT1_DURATION} 分钟）

---

**幻灯片 1 — 标题页**
- 主标题："{FULL_TITLE}"
- 副标题："{SUBTITLE_OR_TAGLINE}"
- 演讲者：{SPEAKER_NAME}
- 活动：{EVENT_NAME} — {MONTH_YEAR}
- 视觉效果：{BACKGROUND_DESCRIPTION}
- 演讲者备注："{OPENING_NOTES}"
- 时长：{TITLE_SLIDE_DURATION} 分钟

---

**幻灯片 2 — {SLIDE2_TITLE}**
- 标题："{SLIDE2_TITLE}"
- 视觉效果：{SLIDE2_VISUAL_DESCRIPTION}
- 关键文字："{SLIDE2_KEY_TEXT}"
- 演讲者备注："{SLIDE2_NOTES}"
- 时长：{SLIDE2_DURATION} 分钟

---

{REPEAT_FOR_REMAINING_SLIDES_IN_ACT1}

---

### 第二幕：{ACT2_TITLE}（幻灯片 {ACT2_FIRST}-{ACT2_LAST}，约 {ACT2_DURATION} 分钟）

---

{SLIDES_FOR_ACT2}

---

### 第三幕：{ACT3_TITLE}（幻灯片 {ACT3_FIRST}-{ACT3_LAST}，约 {ACT3_DURATION} 分钟）

---

{SLIDES_FOR_ACT3}

---

### 第四幕：{ACT4_TITLE}（幻灯片 {ACT4_FIRST}-{ACT4_LAST}，约 {ACT4_DURATION} 分钟）

---

{SLIDES_FOR_ACT4}

---

### 第五幕 + 结语：{ACT5_TITLE}（幻灯片 {ACT5_FIRST}-{LAST_SLIDE}，约 {ACT5_DURATION} 分钟）

---

{SLIDES_FOR_ACT5}

---

## 演讲者备注指南

每张幻灯片的演讲者备注包括：
- 关键叙事节拍（目标引发的情绪/反应）
- 关键时刻的精确措辞（妙语、过渡）
- 时间指引
- 戏剧效果的停顿提示

需要刻意停顿的关键叙事时刻：
1. {PAUSE_MOMENT_1} — {WHY_PAUSE}
2. {PAUSE_MOMENT_2} — {WHY_PAUSE}
3. {PAUSE_MOMENT_3} — {WHY_PAUSE}

## 截图占位符

部分幻灯片设计用于放置真实截图。用标有"截图区域"的占位矩形清晰标注这些区域：

| 幻灯片 | 截图 | 来源 |
|-------|------|------|
| {SLIDE_N} | {SCREENSHOT_DESCRIPTION} | {SOURCE} |
{REPEAT_FOR_ALL_SCREENSHOT_SLIDES}

## 附加要求

- 总幻灯片数：恰好 {SLIDE_COUNT} 张
- 除第1张（标题页）外，所有幻灯片都包含页码
- 在幕与幕之间添加细微的章节过渡标记（角落处的小幕号）
- 全程深色背景（基础 #0a0a0a，卡片 #141414）——投影仪友好
- 所有图表使用强调色配色（深色上的橙色 #f97316，浮层元素用 #1e1e1e）
- 数字是本演示的主角——让它们无法被忽视
- 无动画或复杂转场——幻灯片间简单淡入淡出
- 页脚：右下角低调页码，左下角演讲者姓名
- 整体感觉应为：{TONE_DESCRIPTION}

## 无障碍性

- 确保文字对比度符合 WCAG AA 标准（最低 4.5:1）
- 使用大号可读字体（正文最小 20pt，标题 28pt+，关键数字 48pt+）
- 不要仅依赖颜色传达信息（同时使用图标 + 颜色）
- 在典型投影仪分辨率（1920x1080）下测试可读性

## 语气参考

{TONE_PARAGRAPH}

---

**提示词结束**
