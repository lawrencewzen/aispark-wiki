> 📚 **AI Spark Wiki** · Claude Code 知识库

# Review Comment 模板

使用此模板生成 GitHub PR review 评论。根据 code-reviewer 智能体的输出填写各部分。评论以**英文**发布（面向国际受众）。

---

## 模板

```markdown
## Review

**Scope**: Security, code quality, performance, test coverage, architecture

### Summary

{1–2 sentences: overall assessment. Be direct — what's the main takeaway?}

### Critical Issues

{List blocking issues that must be fixed before merge. For each:}
{- `file:42` — Description of the problem. Why it matters. Suggested fix.}

{If none: "None found."}

### Important Issues

{List significant issues that should be fixed. For each:}
{- `file:42` — Description. Why it matters. Suggested fix.}

{If none: "None found."}

### Suggestions

{List nice-to-haves and minor improvements. For each:}
{- Description. Context. Optional fix.}

{If none: omit this section.}

### What's Good

{Always include at least 1 positive point. Be specific — what works well and why.}
{- Description of what's done right.}

---
*Automated review via Claude Code `/pr-triage` — add your project-specific checks in SKILL.md*
```

---

## 格式规则

**引用格式**：`file:42` 或 `` `code snippet` `` 用于行内引用

**问题严重程度**：
- Critical（严重）：安全漏洞、数据丢失风险、功能损坏、新功能缺少测试
- Important（重要）：错误处理缺失、性能退化、范围蔓延、缺少验证
- Suggestion（建议）：命名、DRY 优化机会、文档、风格

**语气**：专业、建设性、基于事实。针对代码提出质疑，而非针对人。
不使用夸大词语（"great"、"amazing"、"perfect"）。不使用填充语（"as mentioned"、"it's worth noting"）。

**长度**：目标 200–400 词。足够详细有用，足够简洁可读。
