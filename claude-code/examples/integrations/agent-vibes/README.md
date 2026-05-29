> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Agent Vibes TTS - Text-to-Speech for Claude Code"
description: "为 Claude Code 添加文字转语音功能的社区 MCP 服务器"
tags: [mcp, integration, plugin]
---

# Agent Vibes TTS - Claude Code 文字转语音

**状态**：社区 MCP 服务器（可选）
**版本**：3.0.0
**维护者**：[Paul Preibisch](https://github.com/paulpreibisch/AgentVibes)
**许可证**：Apache 2.0

---

## 快速决策矩阵

是否应该安装 Agent Vibes？参考下表：

| 使用场景 | 推荐度 | 原因 |
|----------|----------------|--------|
| 代码审查（边听边多任务处理） | ⭐️⭐️⭐️⭐️⭐️ | 音频朗读解放双眼 |
| 长时间调试会话 | ⭐️⭐️⭐️⭐️ | 音频通知让你保持了解 |
| 高度专注的工作 | ⚠️ 可能令人分心 | 深度工作时考虑静音 |
| 离线 TTS（无网络） | ✅ 完美适合 | Piper TTS 100% 离线 |
| 高品质声音 | ❌ 请用 ElevenLabs | Agent Vibes = 免费，质量尚可 |
| 法语 | ⭐️⭐️⭐️⭐️ | 4 个法语声音（128 个说话人） |
| 英语 | ⭐️⭐️⭐️⭐️⭐️ | 12 个英语声音（高质量） |

---

## 30 秒概览

Agent Vibes 使用以下方式为 Claude Code 响应添加**有声朗读**：

- **15 种声音**（12 英语、4 法语，含 124 个说话人）
- **完全免费**（Piper TTS + macOS Say）
- **完全离线**（无云端依赖）
- **18 分钟安装**（5 个阶段含检查点）
- **34 个斜杠命令**用于语音控制

**安装**：[完整指南](./installation.md) | [快速工作流](../../../guide/workflows/tts-setup.md)
**使用**：[声音目录](./voice-catalog.md) | [故障排除](./troubleshooting.md)

---

## 工作原理

### 架构

```
Claude Code 响应
    ↓
.claude/hooks/play-tts.sh（路由器）
    ↓
[静音检查] → 已静音则退出
    ↓
[提供商管理器]
    ├─ piper   → Piper TTS（离线，神经网络声音）
    ├─ macos   → macOS Say（原生，即时）
    └─ termux  → 通过 SSH 连接 Android
    ↓
[语音处理]
    ├─ 加载声音模型（约 60MB）
    ├─ 生成音频（约 200ms）
    ├─ 应用效果（混响、回声）
    └─ 混入背景音乐（可选）
    ↓
[音频输出] → afplay（macOS）/ 扬声器
```

**性能（M1 Mac）**：
- 音频生成：约 200ms
- 总延迟：约 280ms（生成 + 效果 + 播放）
- 内存占用：约 50MB
- CPU：约 80% 峰值（200ms），随后空闲

---

## 快速开始（5 分钟）

### 前置条件

- 安装了 Homebrew 的 macOS
- Bash 5.x（Agent Vibes 将协助安装）
- Node.js 16+（用于安装）

### 安装

```bash
# 1. 安装 Agent Vibes（交互式，4 页）
npx agentvibes install

# 2. 选择 Piper TTS 提供商（推荐）
# 选择：Piper > fr_FR-tom-medium > 轻混响 > 低冗余度

# 3. 验证安装
ls -la .claude/hooks/play-tts.sh
ls -la ~/.claude/piper-voices/
```

**遇到问题？** 查看[完整安装指南](./installation.md)或[故障排除](./troubleshooting.md)

### 首次测试

```bash
# 启动 Claude Code
claude

# 在 Claude 中测试 TTS
/agent-vibes:whoami
/agent-vibes:list
> "用法语说你好"
```

你应该能听到音频朗读！

---

## 启用与停用

### 快速静音/取消静音

```bash
# 在 Claude Code 中
/agent-vibes:mute      # 静音（跨会话持久）
/agent-vibes:unmute    # 取消静音

# 或手动操作
touch .claude/agentvibes-muted      # 项目静音
touch ~/.agentvibes-muted           # 全局静音
rm .claude/agentvibes-muted         # 项目取消静音
```

### 静音优先级

```
优先级 1：.claude/agentvibes-unmuted  （项目覆盖）
优先级 2：.claude/agentvibes-muted    （项目静音）
优先级 3：~/.agentvibes-muted         （全局静音）
优先级 4：默认激活
```

**示例**：全局静音，对特定项目取消静音：
```bash
touch ~/.agentvibes-muted                 # 所有项目静音
touch .claude/agentvibes-unmuted          # 当前项目取消静音
```

### 完全卸载

```bash
# 自动卸载
npx agentvibes uninstall --yes

# 手动清理（如需要）
rm -rf .claude/hooks/*vibes*
rm -rf .claude/commands/agent-vibes/
rm -rf .claude/audio/
rm -rf ~/.claude/piper-voices/
pipx uninstall piper-tts
```

---

## 核心命令

### 提供商管理

```bash
/agent-vibes:provider list              # 列出可用提供商
/agent-vibes:provider switch piper      # 切换至 Piper TTS
/agent-vibes:provider switch macos      # 切换至 macOS Say
/agent-vibes:provider info              # 当前提供商详情
```

### 声音管理

```bash
/agent-vibes:list                       # 列出所有声音
/agent-vibes:list first 5               # 显示前 5 个声音
/agent-vibes:switch fr_FR-tom-medium    # 切换至法语男声
/agent-vibes:whoami                     # 当前声音和提供商
/agent-vibes:preview                    # 预览前 3 个声音
/agent-vibes:sample fr_FR-tom-medium    # 测试指定声音
```

### 音频控制

```bash
/agent-vibes:mute                       # 静音所有 TTS
/agent-vibes:unmute                     # 取消 TTS 静音
/agent-vibes:replay                     # 重播最后一段音频
/agent-vibes:replay 2                   # 重播倒数第二段
/agent-vibes:verbosity low              # 减少朗读（推荐）
/agent-vibes:verbosity medium           # 增加朗读
```

### 效果与个性化

```bash
/agent-vibes:effects reverb light       # 添加轻混响
/agent-vibes:effects off                # 禁用所有效果
/agent-vibes:background-music on        # 启用背景音乐
/agent-vibes:background-music off       # 禁用背景音乐
/agent-vibes:personality sarcastic      # 切换讽刺个性
/agent-vibes:personality professional   # 专业模式
```

### 工具命令

```bash
/agent-vibes:hide                       # 隐藏全部 34 个命令
/agent-vibes:show                       # 重新显示命令
/agent-vibes:version                    # Agent Vibes 版本
/agent-vibes:update                     # 更新 Agent Vibes
```

**完整命令参考**：共 34 个命令。若命令过多影响命令面板，使用 `/agent-vibes:hide` 隐藏。

---

## 声音目录快速参考

### 法语声音（4 个模型，128 个说话人）

| 声音 | 性别 | 质量 | 说话人数 | 适用场景 |
|-------|--------|---------|----------|----------|
| **fr_FR-tom-medium** | 男 | 中等 | 1 | ⭐️⭐️⭐️⭐️⭐️ 最佳法语男声 |
| fr_FR-siwis-medium | 女 | 中等 | 1 | 清晰自然 |
| fr_FR-upmc-medium | 中性 | 中等 | 1 | 多用途 |
| **fr_FR-mls-medium** | 混合 | 中等 | 124 | ⭐️⭐️⭐️⭐️⭐️ 最大多样性 |

**多说话人**：`fr_FR-mls-medium` 含 124 种不同声音（49 女、75 男）。使用 `-s 0-123` 标志选择特定说话人。

### 英语声音（12 个模型）

| 声音 | 性别 | 质量 | 特点 |
|-------|--------|---------|-----------|
| **en_US-ryan-high** | 男 | 高 | ⭐️⭐️⭐️⭐️⭐️ 专业 |
| en_US-amy-medium | 女 | 中等 | 温暖自然 |
| en_US-lessac-medium | 男 | 中等 | 权威 |
| en_US-libritts-high | 混合 | 高 | 非常自然 |
| ... | ... | ... | 另有 8 种声音 |

**含音频示例的完整目录**：[声音目录](./voice-catalog.md)

---

## 常见使用场景

### 1. 边听边做代码审查

```bash
# 启用低冗余度 TTS
/agent-vibes:verbosity low

# 处理其他任务，同时听 Claude 的分析
> "审查身份验证中间件的安全问题"
```

### 2. 长任务音频通知

```bash
# 运行完整测试套件，完成时收到通知
> "运行完整测试套件并报告失败项"
# → 测试完成时播放音频提醒
```

### 3. 语言学习模式

```bash
# 启用双语 TTS
/agent-vibes:learn on
/agent-vibes:target es_ES
/agent-vibes:target-voice es_ES-davefx-medium

# 响应同时用英语和西班牙语朗读
> "解释依赖注入"
```

### 4. 自定义钩子（仅朗读错误）

```bash
# 创建选择性 TTS 钩子
cat > .claude/hooks/speak-errors-only.sh << 'EOF'
#!/opt/homebrew/bin/bash
INPUT=$(cat)
BODY=$(echo "$INPUT" | jq -r '.notification.body // empty')

# 仅在包含 "error" 或 "failed" 时朗读
if [[ "$BODY" =~ (error|failed|Error|Failed) ]]; then
  ~/.claude/hooks/play-tts.sh "$BODY"
fi
exit 0
EOF

chmod +x .claude/hooks/speak-errors-only.sh
```

---

## 性能优化建议

### 降低延迟

```bash
# 使用低质量声音（生成更快）
/agent-vibes:switch fr_FR-siwis-low  # 如有该版本

# 禁用效果
/agent-vibes:effects off

# 禁用背景音乐
/agent-vibes:background-music off

# 效果：延迟约 150ms 而非约 280ms
```

### 优化电池续航

```bash
# macOS Say（即时，无 CPU 峰值）
/agent-vibes:provider switch macos

# 权衡：质量较低，但生成时间为 0ms
```

### 减少干扰

```bash
# 最低冗余度
/agent-vibes:verbosity low

# 专业个性（话更少）
/agent-vibes:personality professional

# 或在专注工作时静音
/agent-vibes:mute
```

---

## 故障排除

### 无音频

```bash
# 1. 检查静音状态
ls -la .claude/agentvibes-muted ~/.agentvibes-muted

# 2. 验证提供商
cat .claude/tts-provider.txt

# 3. 手动测试 Piper
echo "Test" | piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test.wav && afplay /tmp/test.wav
```

**解决方案**：详细诊断见[故障排除指南](./troubleshooting.md)。

### 声音听起来很机械

**解决方案**：切换至高质量模型：
```bash
# 下载高质量声音
cd ~/.claude/piper-voices
curl -L -o fr_FR-siwis-high.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/high/fr_FR-siwis-high.onnx"

/agent-vibes:switch fr_FR-siwis-high
```

### 34 个命令使命令面板混乱

**解决方案**：隐藏它们：
```bash
/agent-vibes:hide

# 需要时再显示
/agent-vibes:show
```

**更多问题？** 查看[故障排除指南](./troubleshooting.md)

---

## 配置文件

| 文件 | 用途 | 格式 |
|------|---------|--------|
| `.claude/tts-provider.txt` | 当前提供商 | `piper` 或 `macos` 或 `termux` |
| `.claude/tts-voice.txt` | 当前声音 | `fr_FR-tom-medium` |
| `.claude/agentvibes-muted` | 项目静音状态 | 文件存在 = 已静音 |
| `~/.agentvibes-muted` | 全局静音状态 | 文件存在 = 已静音 |
| `.claude/config/audio-effects.cfg` | 音频效果配置 | Key=Value 格式 |
| `~/.claude/piper-voices/*.onnx` | 声音模型 | 神经网络模型 |

---

## 相关文档

- **[安装指南](./installation.md)** - 完整 18 分钟安装流程
- **[声音目录](./voice-catalog.md)** - 全部 15 种声音及音频示例
- **[故障排除](./troubleshooting.md)** - 常见问题与解决方案
- **[TTS 配置工作流](../../../guide/workflows/tts-setup.md)** - 逐步安装指南
- **[AI 生态系统](../../../guide/ecosystem/ai-ecosystem.md#47-voice-interfaces)** - AI 上下文中的 TTS

---

## 资源

- **GitHub**：https://github.com/paulpreibisch/AgentVibes
- **官网**：https://agentvibes.org
- **演示视频**：https://youtu.be/ngLiA_KQtTA
- **Piper 声音**：https://huggingface.co/rhasspy/piper-voices
- **Piper 示例**：https://rhasspy.github.io/piper-samples/

---

## 贡献

Agent Vibes 是一个社区项目。提交问题或参与贡献：
- **Issues**：https://github.com/paulpreibisch/AgentVibes/issues
- **许可证**：Apache 2.0

---

*集成指南由 [Claude Code Ultimate Guide](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide) 维护*
*最后更新：2026-01-22 | Agent Vibes v3.0.0 | Piper TTS v1.3.0*
