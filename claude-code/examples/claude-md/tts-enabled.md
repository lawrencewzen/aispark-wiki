> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "启用 TTS 的项目模板"
description: "使用 Agent Vibes 文字转语音的项目 CLAUDE.md 配置"
tags: [claude-md, template, tts]
---

# 启用 TTS 的项目

这是使用 Agent Vibes TTS 的项目所需的 `CLAUDE.md` 模板文件。

## TTS 配置

**提供商**：Piper TTS
**语音**：fr_FR-tom-medium（法语男声）
**详细程度**：低（推荐）
**音效**：轻微混响
**背景音乐**：已禁用

## 项目专属 TTS 设置

### 专注工作时静音

在需要深度专注的任务期间，将 TTS 静音：

```bash
# 本次会话静音
/agent-vibes:mute

# 完成后取消静音
/agent-vibes:unmute
```

### 选择性 TTS（仅错误）

本项目的 TTS 仅朗读错误信息：

- ✅ 错误、失败、异常
- ❌ 普通响应、确认提示
- ❌ 普通信息提示

**原因**：这是一个关键生产系统，错误的音频提醒很有价值，但持续旁白会干扰工作。

## 语音偏好

| 任务类型 | 推荐语音 | 原因 |
|-----------|-------------------|--------|
| 代码审查 | fr_FR-tom-medium | 专业、清晰 |
| 文档编写 | fr_FR-siwis-medium | 温和、有教学感 |
| 调试 | fr_FR-tom-medium（低详细度） | 仅关键告警 |

## 命令速查

团队成员快速参考：

```bash
# 查看当前语音
/agent-vibes:whoami

# 切换语音
/agent-vibes:switch fr_FR-tom-medium

# 静音/取消静音
/agent-vibes:mute
/agent-vibes:unmute

# 调整详细程度
/agent-vibes:verbosity low

# 关闭音效（速度更快）
/agent-vibes:effects off
```

## 团队使用指南

### 何时静音

- 结对编程时（讲解者说话，TTS 会干扰）
- 视频会议时（避免音频冲突）
- 深度专注工作时（优先保持心流）
- 公共场所时（避免打扰他人）

### 何时开启

- 独自做代码审查时（边看 diff 边听）
- 长时间任务时（音频完成通知）
- 后台监控时（错误告警）
- 学习模式时（双语练习）

## 新成员安装指南

新团队成员请参照以下步骤：

1. **安装指南**：[Agent Vibes 安装说明](../integrations/agent-vibes/installation.md)
2. **语音选择**：统一使用 `fr_FR-tom-medium`
3. **配置**：从本文件复制设置

## 故障排查

常见问题：

- **无声音**：检查 `cat .claude/tts-provider.txt`——应显示 `piper`
- **语音错误**：运行 `/agent-vibes:switch fr_FR-tom-medium`
- **过于啰嗦**：运行 `/agent-vibes:verbosity low`

**完整故障排查**：[Agent Vibes 故障排查](../integrations/agent-vibes/troubleshooting.md)

## 说明

- TTS 是**可选的**——不是参与项目贡献的必要条件
- 静音状态是**项目级别**的（`.claude/agentvibes-muted`）
- 语音模型是**全局共享**的（`~/.claude/piper-voices/`）

---

*更多关于 Agent Vibes TTS 的信息，请参阅 [集成指南](../integrations/agent-vibes/README.md)*
