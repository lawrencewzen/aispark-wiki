> 📚 **AI Spark Wiki** · Claude Code 知识库

# Twitter/X 模板

两种模式：单条推文（280 字符）或话题串（2-3 条推文）。

## 单条推文

适用场景：1-2 个亮点，简单版本更新。

### FR 模板

```
{hook_line_fr}

{highlight_fr}

{link}
```

### EN 模板

```
{hook_line_en}

{highlight_en}

{link}
```

### 规则

- 总计最多 280 字符（含链接）
- 最多 2 个 emoji
- 链接至 GitHub 仓库
- FR：使用非正式称谓（tutoiement）
- EN：直接称呼

## 话题串（2-3 条推文）

适用场景：3 个及以上亮点，内容丰富的版本/周报。

### FR 模板

```
Tweet 1/N:
{hook_line_fr}

{context_fr}

Thread [down_arrow]

---

Tweet 2/N:
{highlight_1_fr}
{highlight_2_fr}

---

Tweet 3/N（可选）:
{highlight_3_fr}

{cta_fr}
{link}
```

### EN 模板

```
Tweet 1/N:
{hook_line_en}

{context_en}

Thread [down_arrow]

---

Tweet 2/N:
{highlight_1_en}
{highlight_2_en}

---

Tweet 3/N (optional):
{highlight_3_en}

{cta_en}
{link}
```

## 字段规则

### hook_line（最多 100 字符）

顶部亮点的最简表达形式，须与上下文一起放入第一条推文。

| 模式 | FR | EN |
|---------|-----|-----|
| 数字开头 | `30 nouvelles questions quiz Claude Code` | `30 new Claude Code quiz questions` |
| 直接陈述 | `Nouveau guide : sandbox isolation Docker` | `New guide: Docker sandbox isolation` |

### context（最多 80 字符）

```
FR: "Guide v3.20.5 vient de sortir"
EN: "Guide v3.20.5 just dropped"
```

### highlights（每条最多 120 字符）

经过转化的条目，每条一行，不使用列表符号（用换行分隔）。

```
FR: "4 diagrammes ASCII pour TDD, UVAL, securite"
EN: "4 ASCII diagrams for TDD, UVAL, security"
```

### cta（最多 60 字符）

```
FR: "Tout est open source"
EN: "All open source"
```

### link

GitHub 仓库 URL。计入 280 字符限制（t.co 短链占 23 字符）。

## 决策：单条 vs 话题串

| 条件 | 格式 |
|-----------|--------|
| 1-2 个亮点，全部内容在 280 字符内 | 单条推文 |
| 3 个及以上亮点或内容丰富 | 话题串（2-3 条） |
| 包含多个版本的周报 | 话题串 |
| 仅为维护性变更 | 单条推文（或跳过） |

## 约束

- 每条推文：最多 280 字符
- Emoji 预算：整个话题串总计 2 个
- 不使用夸张词汇
- FR：使用非正式称谓（tutoiement）
- EN：美式英语
- 话题串上限：3 条（不得超过 5 条）
