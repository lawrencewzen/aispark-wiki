> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: voice-refine
description: "将冗长的语音输入转换为结构化、节省 token 的 Claude 提示词。适用于清理语音备忘录、口述内容或语音转文字输出，这些内容通常包含填充词、重复和无结构的想法。"
allowed-tools: Read
effort: low
---

# Voice Refine Skill

将冗长的意识流语音口述转换为结构化、节省 token 的 Claude Code 提示词。

## 适用场景

- 来自语音口述的输入（Wispr Flow、Superwhisper、macOS Dictation）
- 超过 150 字的冗长文本
- 包含填充词、重复或跑题内容
- 需要结构化的自然口语模式

## 转换流水线

```
1. DEDUPE    → 去除重复和填充词
2. EXTRACT   → 提取核心需求与约束
3. STRUCTURE → 组织为标准章节
4. COMPRESS  → 在保留意图的前提下压缩至原文的约 30%
```

## 输出格式

```markdown
## Contexte
[项目背景、现有技术栈、相关文件]

## Objectif
[一句话：需要构建/修改什么]

## Contraintes
- [约束 1]
- [约束 2]
- [以此类推]

## Output attendu
[预期交付物：文件、格式、测试]
```

## 标志位

| 标志 | 效果 |
|------|--------|
| `--confirm` | 发送给 Claude 前显示精炼后的提示词（默认） |
| `--direct` | 不经确认直接发送精炼后的提示词 |
| `--verbose` | 保留更多细节，减少压缩 |
| `--en` | 输出英文（默认：与输入语言一致） |

## 使用示例

### 基础用法

```
/voice-refine

Alors euh j'aimerais que tu m'aides à faire un truc, en fait j'ai une API
qui renvoie des données utilisateurs et je voudrais les afficher dans un
tableau React, mais attention il faut que ça soit paginé parce que y'a
beaucoup de données, genre des milliers d'utilisateurs, et aussi faudrait
pouvoir trier par nom ou par date d'inscription, ah et on utilise Tailwind
dans le projet donc faut que ça matche avec ça...
```

### 带标志位

```
/voice-refine --direct --en

[任意语言的语音输入 → 直接发送英文提示词]
```

## 压缩指标

| 指标 | 目标 |
|--------|--------|
| Token 缩减率 | 60-70% |
| 信息保留率 | >95% |
| 结构清晰度 | 高 |

## 过滤规则

**去除**：填充词（"euh"、"um"、"like"、"basically"）、重复内容、跑题内容、不确定性表达（"maybe"、"probably"，除非与内容相关）、礼貌性填充词（"please"、"could you"）。

**保留**：技术需求、约束条件、现有代码上下文、预期输出格式、边界情况、业务逻辑规则。

## 参见

- `guide/ai-ecosystem.md` — 语音转文字工具章节
- `examples/before-after.md` — 完整转换示例
