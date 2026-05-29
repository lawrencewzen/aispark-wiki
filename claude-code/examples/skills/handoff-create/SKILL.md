> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: handoff-create
description: 从当前会话生成结构化的交接文档。捕获范围、含行号的相关文件、关键发现、已完成工作、当前状态、后续步骤和代码片段。在结束会话或将工作交接给其他智能体前使用。
argument-hint: "[optional-filename]"
effort: low
disable-model-invocation: true
---

从我们当前的对话生成结构化的交接文档，并保存到 `claudedocs/handoffs/handoff_YYYYMMDD_HHMMSS.md`（如果通过 `$ARGUMENTS[0]` 提供了文件名，则使用该文件名）。

如果 `claudedocs/handoffs/` 目录不存在，请先创建它。

## 文档结构

必须包含以下章节：

```markdown
# Handoff — [Task Name] — [YYYY-MM-DD HH:MM]

## Task

[一句话：正在完成什么]

## Scope

[需要做什么。明确说明边界——什么在范围内，什么明确不在范围内]

## Files

[所有涉及或需要的相关文件。当某行特别重要时，格式为：`path/to/file:line`]

- `src/auth/middleware.ts:45` — token 验证逻辑
- `tests/auth.spec.ts` — 本次改动的测试套件
- `docs/api.md` — 实现后需要更新

## Discoveries

[到目前为止工作中的关键发现和洞察。下一个智能体需要了解的、从代码中无法显见的内容]

- [发现 1]
- [发现 2]

## Work Done

[已完成的任务。仅追加——永不删除已有条目。如有提交哈希请附上]

- [x] 任务 A 已完成（commit: abc1234）
- [x] 任务 B 已完成

## Status

[当前状态：已完成什么、还剩什么、测试状态、阻塞项]

## Next Steps

[按顺序排列的剩余任务可操作清单]

1. [ ] 步骤 1
2. [ ] 步骤 2
3. [ ] 步骤 3

## Code

[带上下文的相关代码片段。聚焦于接手的智能体需要立即理解的部分]

\`\`\`typescript
// 关键实现细节
\`\`\`
```

## 规则

- 总字数控制在 600 字以内。接手的智能体读完此文档后应在 30 秒内能够继续工作。
- 所有文件引用使用 `path/to/file:line` 格式，行号很重要。
- "已完成工作"仅追加。永不删除已有条目。
- 不要包含完整的 git diff 或完整的文件内容。只包含不显而易见的代码片段。

保存后，确认文件路径。
