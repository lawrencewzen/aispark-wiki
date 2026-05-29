> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "AI 使用声明（CONTRIBUTING.md 模板）"
description: "用于在 PR 中声明 AI 工具使用情况的 CONTRIBUTING.md 模板章节"
tags: [template, config, ai-ecosystem]
---

# AI 使用声明（CONTRIBUTING.md 模板）

> 将此章节复制到你项目的 CONTRIBUTING.md 中

---

## AI 使用声明

如果你在贡献过程中使用了任何 AI 工具，请在 pull request 描述中说明。

### 需要声明的内容

| AI 使用方式 | 声明示例 |
|------------|---------|
| **AI 生成代码** | "本 PR 主要由 Claude Code 编写" |
| **AI 辅助调研** | "我使用 ChatGPT 辅助理解代码库" |
| **AI 建议方案** | "算法结构由 Copilot 建议" |
| **AI 起草文档** | "文档由 Claude 辅助起草" |

### 无需声明的内容

- 简单自动补全（单个关键词、短语）
- IDE 语法辅助（格式化、自动导入）
- 语法/拼写检查
- 代码格式化工具（prettier、black）

### 为何要求声明

AI 生成的代码通常需要更仔细的审查：

- 可能使用代码库中不熟悉的模式
- 可能引入人类不会犯的细微 bug
- 可能忽略项目特定的约定
- 有时"看起来正确"但存在逻辑问题

声明有助于维护者：
- 合理分配审查时间
- 知道哪里需要重点关注
- 对 AI 使用给出更好的反馈

这是**对审查者的一种尊重**，而非对 AI 使用的评判。

### 推荐声明格式

在 PR 描述中：

```markdown
## AI Assistance

This PR was developed with assistance from [Tool Name].
Specifically:
- [What AI helped with]
- [What you did manually]

All code has been reviewed and understood by the author.
```

---

## 参考来源

参考以下项目的相关政策：
- [Ghostty](https://github.com/ghostty-org/ghostty/blob/main/CONTRIBUTING.md)
- [LLVM](https://llvm.org/docs/DeveloperPolicy.html)
- [Fedora](https://docs.fedoraproject.org/en-US/project/ai-policy/)

更多背景参见 [AI 可追溯性指南](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide/blob/main/guide/ops/ai-traceability.md)。
