> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "行为模式"
description: "用于自定义 Claude Code 行为的即用型行为模式文件"
tags: [config, template, workflows]
---

# 行为模式

适用于 Claude Code 的即用型行为模式文件。将其复制到 `~/.claude/` 目录并在 `CLAUDE.md` 中引用。

## 可用模式

| 模式 | 文件 | 用途 |
|------|------|------|
| **学习模式** | [MODE_Learning.md](./MODE_Learning.md) | 首次使用某技术时提供即时说明 |

## 安装

### 1. 复制模式文件

```bash
cp MODE_Learning.md ~/.claude/
```

### 2. 在 CLAUDE.md 中引用

在 `~/.claude/CLAUDE.md` 中添加：

```markdown
# Behavioral Modes
@MODE_Learning.md
```

### 3. 添加标志（可选）

在 `~/.claude/FLAGS.md` 中添加，以支持基于标志的激活：

```markdown
**--learn**
- Trigger: User requests learning mode, "why/how" questions
- Behavior: Enable just-in-time explanations with first-occurrence tracking

**--no-learn**
- Trigger: User wants pure execution without educational offers
- Behavior: Suppress all learning mode offers
```

## 用法

```bash
# 在整个会话中激活
claude --learn

# 聚焦于特定领域
claude --learn focus:git
claude --learn focus:architecture

# 在任务结束时批量说明
claude --learn batch
```

## 更多模式：SuperClaude 框架

本指南仅包含**学习模式**。如需包含更多模式的完整行为框架，请参阅 [SuperClaude](https://github.com/SuperClaude-Org/SuperClaude_Framework)：

| 模式 | 用途 |
|------|------|
| **编排模式** | 智能工具选择与并行执行优化 |
| **任务管理模式** | 带持久记忆的层级任务追踪 |
| **Token 效率模式** | 符号增强压缩（减少 30-50% token） |
| **学习模式** | 即时技能培养（已包含在本指南中） |

SuperClaude 还包括：
- `FLAGS.md` — 行为标志（`--delegate`、`--learn` 等）*注意：`--think`/`--ultrathink` 自 v2.0.67 起仅为视觉标记——Opus 4.5 现已默认开启思考*
- `PRINCIPLES.md` — 工程原则（SOLID、DRY、循证驱动）
- `RULES.md` — 带优先级系统的可执行规则
- MCP 服务器文档（Context7、Sequential、Serena）

## 参见

- [指南第 10.5 节：SuperClaude 框架](../../guide/ultimate-guide.md#105-superclaude-framework) — 完整文档
- [SuperClaude 代码库](https://github.com/SuperClaude-Org/SuperClaude_Framework) — 完整框架
