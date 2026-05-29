> 📚 **AI Spark Wiki** · Claude Code 知识库

# Slack 模板

精简、易读、富含 emoji，可直接粘贴使用。

## 中文模板

```
:newspaper: *{title_zh}*

{highlights_zh}

:link: {link}
```

## 英文模板

```
:newspaper: *{title_en}*

{highlights_en}

:link: {link}
```

## 字段规则

### title（最多 60 个字符）

```
ZH: "指南 v3.20.5 - 视觉参考"
EN: "Guide v3.20.5 - Visual reference"

ZH: "本周回顾：4 个版本发布"
EN: "Week recap: 4 releases"
```

### highlights（3-5 行）

每行：Slack emoji + 简短描述。正文不加粗。

```
ZH:
:art: 4 张新 ASCII 示意图（TDD、UVAL、安全、事故）
:brain: 30 道新测验题（共 257 道）
:shield: Docker 沙盒隔离指南
:mag: 通过 claudelog.com 识别的 9 种高级模式

EN:
:art: 4 new ASCII diagrams (TDD, UVAL, security, incidents)
:brain: 30 new quiz questions (257 total)
:shield: Docker sandbox isolation guide
:mag: 9 advanced patterns identified via claudelog.com
```

### link

GitHub 代码库 URL。

## Slack Emoji 参考

使用在所有工作区都能正常渲染的标准 Slack emoji：

| Emoji | 代码 | 适用场景 |
|-------|------|---------|
| :newspaper: | `:newspaper:` | 标题标记 |
| :art: | `:art:` | 视觉内容、示意图、UI |
| :brain: | `:brain:` | 测验、学习、知识 |
| :shield: | `:shield:` | 安全相关内容 |
| :mag: | `:mag:` | 研究、分析、竞品情报 |
| :wrench: | `:wrench:` | 工具、工作流、配置 |
| :books: | `:books:` | 新指南、文档 |
| :chart_with_upwards_trend: | `:chart_with_upwards_trend:` | 增长指标 |
| :white_check_mark: | `:white_check_mark:` | 修复、纠正 |
| :link: | `:link:` | 链接标记 |
| :arrow_right: | `:arrow_right:` | 增长指示（X -> Y） |

## 限制

- 总字符数不超过 500
- emoji 用量：4-6 个（标题 1 个 + 每条要点 1 个 + 链接 1 个）
- 禁用夸张词汇
- 中文：使用口语化表达
- 英文：美式英语
- 仅一个链接（GitHub）
- 禁用话题标签（不符合 Slack 惯例）
- 使用 Slack 格式：`*粗体*`、`_斜体_`、`:emoji:` 代码
