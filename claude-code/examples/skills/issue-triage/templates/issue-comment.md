> 📚 **AI Spark Wiki** · Claude Code 知识库

# Issue 评论模板

在 `/issue-triage` 第 3 阶段，使用以下模板生成 GitHub Issue 评论。评论使用**英文**发布（面向国际受众）。

---

## 模板 1 — 确认收到 / 信息补充请求

适用场景：Issue 状态为 `Unclear` 或 `needs-info`、正文缺少上下文、未提供复现步骤。

```markdown
Thanks for opening this issue!

To help us investigate, could you provide the following?

- **Steps to reproduce**: A minimal sequence of actions that triggers the behavior
- **Expected behavior**: What you expected to happen
- **Actual behavior**: What actually happened
- **Environment**: OS, version, relevant config (if applicable)

Once we have this information, we can prioritize and route the issue appropriately.

---
*Triaged via Claude Code /issue-triage*
```

---

## 模板 2 — 重复 Issue

适用场景：与现有开放或近期关闭的 Issue 的 Jaccard 相似度 >= 60%。

```markdown
Thanks for reporting this! After reviewing the backlog, this appears to be a duplicate of #{original_number}.

{original_number} is tracking the same underlying behavior: {one-sentence description of original}.

I'm closing this issue to consolidate discussion there. If you believe this is a distinct issue with different root cause or scope, please reopen with additional context explaining the difference.

---
*Triaged via Claude Code /issue-triage*
```

---

## 模板 3 — 关闭陈旧 Issue

适用场景：Issue 超过 90 天无活动且无负责人，或对先前信息请求超过 30 天无响应。

```markdown
This issue has been inactive for {days} days without updates or response.

We're closing it to keep the backlog actionable. If this is still relevant to you, please reopen and provide:

- Current status: is this still reproducible?
- Any additional context or workarounds you've found

We're happy to pick this back up if it's still blocking you.

---
*Triaged via Claude Code /issue-triage*
```

---

## 模板 4 — 关闭超出范围的 Issue

适用场景：Issue 描述的功能明显超出项目既定范围。

```markdown
Thanks for the suggestion! After review, this falls outside the current scope of this project.

{Briefly explain why — e.g., "This project focuses on X; Y is handled by Z" or "This would require changes to the underlying architecture that are not planned."}

You might find what you're looking for in:
- {alternative project or tool if known}
- {documentation link if relevant}

Feel free to open a discussion if you'd like to explore this further.

---
*Triaged via Claude Code /issue-triage*
```

---

## 格式规范

**语气**：专业、直接、尊重。提交者花时间提交了 Issue。

**规则**：
- 不暗示提交者有错或浪费了时间
- 引用重复 Issue 时要具体（标题 + 编号，不能只有编号）
- 关闭陈旧 Issue 时：始终邀请重新打开——不要让人感觉是永久关闭
- 关闭超范围 Issue 时：尽可能提供替代方案，哪怕比较模糊
- 不使用溢美之词（"great issue"、"awesome report"）——只陈述事实

**自定义字段**（在所有模板中替换）：
- `{original_number}`：原始重复 Issue 的编号
- `{one-sentence description}`：原始 Issue 的简短摘要
- `{days}`：自最后活动以来的天数（来自 `updatedAt`）
- 用 `{...}` 标注的内联说明：发布前必须填写

**签名**（所有评论必须附带）：
```
*Triaged via Claude Code /issue-triage*
```
