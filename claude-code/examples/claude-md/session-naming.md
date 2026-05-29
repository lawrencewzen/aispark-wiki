> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "会话自动重命名"
description: "CLAUDE.md 片段：让 Claude 在 2-3 轮交互后自动将会话命名为描述性标题"
tags: [session, resume, productivity, workflow]
---

# 会话自动重命名 — CLAUDE.md 片段

将以下代码块添加到全局 `~/.claude/CLAUDE.md`，让 Claude 在 2-3 轮交互后自动为会话命名为描述性标题。在并行运行多个会话时（WebStorm、分屏终端、多项目）极为有用。

## 问题所在

同时运行多个 Claude Code 会话时，它们在会话选择器中都显示为"claude"或截断的首条提示词。找到正确的会话来 `/resume` 完全靠猜。

## 解决方案

在 CLAUDE.md 中加入一条行为指令——无需脚本、无需钩子、无需插件。Claude 会在早期理解会话主题，并主动调用 `/rename`。

## 片段

```markdown
# Session Naming (auto-rename)

## Expected behavior

1. **Early rename**: Once the session's main subject is clear (after 2-3 exchanges),
   run `/rename` with a short, descriptive title (max 50 chars)
2. **End-of-session update**: If scope shifted significantly from the initial rename,
   propose a re-rename before closing

## Title format

`[action] [subject]` — examples:
- "fix whitepaper PDF build"
- "add auth middleware + tests"
- "refactor hook system"
- "research terminal tab rename"
- "update CC releases v2.2.0"

## Rules

- Max 50 characters
- No "Session:" prefix, no date
- Action verb first (fix, add, refactor, update, research, debug...)
- Multi-topic: use the dominant subject, not an exhaustive list
- Do NOT ask for confirmation on the early rename (just do it)
- Only propose confirmation for end-of-session re-rename if title changed
```

## 使用方式

**全局**（所有项目）：添加到 `~/.claude/CLAUDE.md`

**项目级别**：添加到 `.claude/CLAUDE.md` 或项目根目录下的 `CLAUDE.md`

## 工作原理

这是一条纯行为指令——无需任何工具。Claude 会：
1. 从前 2-3 轮交互中推断会话主题
2. 自动调用 `/rename "fix auth middleware"`（无需确认提示）
3. 如果工作重心发生显著偏移，在会话结束时提议重新命名

命名后的会话会在 `/resume` 选择器中以描述性标题显示，便于快速找到并继续正确的会话。

## 局限性

- **标签页重命名**：终端标签页名称（WebStorm、iTerm2）**不会**被重命名。JetBrains 会过滤用于更改标签页标题的 ANSI 转义序列。Claude 会话本身会被重命名，但不影响终端标签页。
- **时机**：Claude 在理解主题后才会重命名，而非在收到第一条消息时立即执行。

## 验证

```bash
# 配置完成后开启一个会话：
claude --resume
# → 会话列表显示描述性名称，如 "fix auth middleware"
# 而非时间戳或截断的提示词
```

## 方案对比

| 方案 | 优点 | 缺点 |
|------|------|------|
| 本方案（行为指令） | 零工具依赖，到处适用 | Claude 须自行判断时机 |
| 手动 `/rename` | 完全可控 | 需要用户操作 |
| 钩子（Stop 事件） | 全自动 | 无法访问对话上下文 |

行为指令在简洁性和可移植性上优势明显。
