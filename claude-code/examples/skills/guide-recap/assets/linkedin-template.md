> 📚 **AI Spark Wiki** · Claude Code 知识库

# LinkedIn 模板

目标：约 1300 个字符。结构：钩子 + 背景 + 要点列表 + 行动号召 + 话题标签。

## 法语模板

```
{hook_line_fr}

{context_line_fr}

{bullet_1_fr}
{bullet_2_fr}
{bullet_3_fr}

{cta_fr}

#ClaudeCode #CodingWithAI #DeveloperTools
```

## 英语模板

```
{hook_line_en}

{context_line_en}

{bullet_1_en}
{bullet_2_en}
{bullet_3_en}

{cta_en}

#ClaudeCode #CodingWithAI #DeveloperTools
```

## 字段规则

### hook_line（1 行，最多 150 个字符）

得分最高的亮点，转化为用户价值。可包含 0-1 个 emoji。

| 模式 | 法语示例 | 英语示例 |
|---------|------------|------------|
| 数字开头 | `30 nouvelles questions quiz pour tester vos connaissances Claude Code` | `30 new quiz questions to test your Claude Code knowledge` |
| 疑问句 | `Vous apprenez mieux en visuel ? 4 nouveaux diagrammes ASCII` | `Visual learner? 4 new ASCII diagrams just added` |
| 来源开头 | `Pat Cullen partage son workflow de code review multi-agent` | `Pat Cullen shares his multi-agent code review workflow` |

### context_line（1-2 行，最多 200 个字符）

版本引用 + 高层次变更概述。

```
FR: "Guide v3.20.5 - mise a jour de la reference visuelle."
EN: "Guide v3.20.5 - visual reference update."
```

周报模式：
```
FR: "N releases cette semaine dans le Claude Code Ultimate Guide."
EN: "N releases this week in the Claude Code Ultimate Guide."
```

### bullets（3 条，每条最多 200 个字符）

按得分排列的前 3 个亮点。每条以相关 emoji 开头（每条最多 1 个）。

```
FR:
- [emoji] [以 vous 形式表达的转化亮点]
- [emoji] [以 vous 形式表达的转化亮点]
- [emoji] [以 vous 形式表达的转化亮点]

EN:
- [emoji] [直接"you"称谓的转化亮点]
- [emoji] [直接"you"称谓的转化亮点]
- [emoji] [直接"you"称谓的转化亮点]
```

### cta（1 行，最多 150 个字符）

温和的价值陈述或真诚的问题。链接至落地页。

```
FR: "Guide complet disponible en open source : {landing_url}"
EN: "Full guide available open source: {landing_url}"
```

### hashtags（1 行，恰好 3 个）

固定：`#ClaudeCode #CodingWithAI #DeveloperTools`

如果话题明确相关，可添加 1 个专题标签：`#CodeReview`、`#Security`、`#TDD`

## 约束条件

- 帖子总长：1100-1500 个字符
- Emoji 预算：总计 3-4 个（钩子 0-1、每条要点 1 个、行动号召 0-1）
- 不使用夸大词汇（参见 tone-guidelines.md）
- 法语：使用 vouvoiement（正式您称）
- 英语：美式英语
