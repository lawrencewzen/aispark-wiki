> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Agent Vibes TTS - 完整安装指南"
description: "在 macOS 上安装 Agent Vibes 文字转语音集成的分步安装指南"
tags: [guide, tts, integration]
---

# Agent Vibes TTS - 完整安装指南

**所需时间**：约 18 分钟
**难度**：中级
**系统**：macOS（Apple Silicon 或 Intel）

---

## 前提条件检查

开始前，请确认以下环境已就绪：

| 要求 | 检查命令 | 最低版本 |
|------|----------|----------|
| **macOS** | `sw_vers` | 10.15+ |
| **Homebrew** | `brew --version` | 任意近期版本 |
| **Node.js** | `node --version` | 16.0.0+ |
| **Python** | `python3 --version` | 3.10.0+ |
| **Git** | `git --version` | 任意近期版本 |

---

## 安装概览（5 个阶段）

```
阶段 1：系统依赖        (~5 分钟)
    ├─ Bash 5.x
    ├─ sox, ffmpeg, util-linux
    └─ espeak-ng

阶段 2：Agent Vibes 安装  (~5 分钟)
    ├─ 交互式安装器
    ├─ 提供商选择（Piper）
    ├─ 语音选择
    └─ 配置

阶段 3：Piper TTS + 语音   (~5 分钟)
    ├─ 通过 pipx 安装 Piper
    ├─ 下载法语语音（4 个）
    └─ 下载英语语音（12 个）

阶段 4：配置             (~2 分钟)
    ├─ 设置提供商：piper
    ├─ 设置语音：fr_FR-tom-medium
    └─ 测试音频

阶段 5：验证             (~1 分钟)
    ├─ 在 Claude Code 中测试
    └─ 确认钩子已激活
```

---

## 阶段 1：系统依赖

### 步骤 1.1：安装 Bash 5.x

Agent Vibes 需要 Bash 5.x（macOS 自带的是 3.2 版本）。

```bash
# 安装 Bash 5.x
brew install bash

# 验证安装
/opt/homebrew/bin/bash --version
# 预期输出：GNU bash, version 5.x
```

**原因**：Agent Vibes 脚本使用了 Bash 5.x 的特性（关联数组等）。

### 步骤 1.2：安装音频工具

```bash
# 安装音频处理工具
brew install sox ffmpeg util-linux

# 验证 sox
sox --version

# 验证 ffmpeg
ffmpeg -version

# 验证 flock（来自 util-linux）
/opt/homebrew/opt/util-linux/bin/flock --version
```

**注意**：`util-linux` 是"keg-only"模式（未创建符号链接），但 Agent Vibes 会自动找到它。

### 步骤 1.3：安装 espeak-ng（Piper 依赖）

```bash
# 安装 espeak-ng
brew install espeak-ng

# 验证安装
espeak-ng --version
```

**原因**：Piper TTS 需要 `libespeak-ng` 库来处理音素。

### 检查点 1：依赖安装完成 ✅

```bash
# 验证所有依赖
command -v /opt/homebrew/bin/bash && \
command -v sox && \
command -v ffmpeg && \
command -v espeak-ng && \
echo "✅ All dependencies installed"
```

---

## 阶段 2：Agent Vibes 安装

### 步骤 2.1：启动交互式安装器

```bash
# 进入你的 Claude Code 项目目录
cd /path/to/your/project

# 启动安装器（交互式，共 4 页）
npx agentvibes install
```

**预期效果**：显示 ASCII 艺术横幅和欢迎界面。

### 步骤 2.2：浏览安装页面

**第 1/4 页：系统依赖**
- 查看检测到的依赖
- 所有项目应显示 `✓`（绿色勾选）
- 点击 **"Next →"**

**第 2/4 页：提供商选择**
- 选项：`macOS Say`、`Piper TTS`、`Termux SSH`
- **选择**：`Piper TTS`（质量最佳，支持离线）
- 点击 **"Next →"**

**第 3/4 页：语音选择**
- **法语**：选择 `fr_FR-tom-medium`（男声）或 `fr_FR-siwis-medium`（女声）
- **英语**：选择 `en_US-ryan-high`（最高质量）
- 点击 **"Next →"**

**第 4/4 页：音频设置**
- **混响**：选择 `Light`（推荐）
- **背景音乐**：选择 `Disabled`（避免分心）
- **详细程度**：选择 `Low`（减少语音提示）
- 点击 **"Start Installation"**

### 步骤 2.3：安装进度

Agent Vibes 将安装以下内容：
- 34 个 slash 命令
- TTS 脚本（40 个 Bash 脚本）
- 个性模板
- 16 个背景音乐曲目
- 7 个配置文件

**预期输出**：
```
✔ Installed 34 slash commands!
✔ Installed TTS scripts!
✔ Installed personality templates!
✔ Installed 16 background music tracks!
✔ Installed 7 config files!

✅ AgentVibes is Ready!
```

### 检查点 2：Agent Vibes 安装完成 ✅

```bash
# 验证安装
ls -la .claude/hooks/play-tts.sh
ls -la .claude/commands/agent-vibes/
ls -la .claude/audio/tracks/

# 检查提供商配置
cat .claude/tts-provider.txt
# 预期："macos" 或 "piper"
```

---

## 阶段 3：Piper TTS + 语音模型

### 步骤 3.1：通过 pipx 安装 Piper TTS

如果选择了 Piper 提供商，Agent Vibes 会尝试自动安装。若安装失败，请手动安装：

```bash
# 通过 pipx 安装 Piper（Python 包管理器）
pipx install piper-tts

# 验证安装
piper --help
```

**常见问题**：预编译二进制文件报 `libespeak-ng.1.dylib` 错误。

**解决方案**：`pipx install piper-tts` 更可靠（Python 版本，非二进制版本）。

### 步骤 3.2：下载法语语音

```bash
# 进入语音存储目录
cd ~/.claude/piper-voices

# 下载 4 个法语语音模型
curl -L -o fr_FR-tom-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx"
curl -L -o fr_FR-tom-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx.json"

curl -L -o fr_FR-siwis-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/medium/fr_FR-siwis-medium.onnx"
curl -L -o fr_FR-siwis-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/medium/fr_FR-siwis-medium.onnx.json"

curl -L -o fr_FR-upmc-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/upmc/medium/fr_FR-upmc-medium.onnx"
curl -L -o fr_FR-upmc-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/upmc/medium/fr_FR-upmc-medium.onnx.json"

curl -L -o fr_FR-mls-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/mls/medium/fr_FR-mls-medium.onnx"
curl -L -o fr_FR-mls-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/mls/medium/fr_FR-mls-medium.onnx.json"
```

**大小**：每个语音约 60-73MB（4 个语音共约 260MB）

### 步骤 3.3：下载英语语音（可选）

Agent Vibes 在安装过程中会自动下载 12 个英语语音。验证：

```bash
ls ~/.claude/piper-voices/en_US-*.onnx
```

**预期**：12 个文件（ryan、amy、lessac、bryce 等）

### 检查点 3：语音下载完成 ✅

```bash
# 统计语音数量
ls ~/.claude/piper-voices/*.onnx | wc -l
# 预期：15 个以上文件（最少 12 个英语 + 4 个法语）

# 手动测试法语语音
echo "Bonjour, je suis Claude et je parle français" | \
  piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test-fr.wav && afplay /tmp/test-fr.wav
```

---

## 阶段 4：配置

### 步骤 4.1：设置 Piper 为提供商

```bash
# 切换至 Piper TTS（若尚未设置）
echo "piper" > .claude/tts-provider.txt

# 验证
cat .claude/tts-provider.txt
# 预期："piper"
```

### 步骤 4.2：设置法语男声

```bash
# 设置默认语音
echo "fr_FR-tom-medium" > .claude/tts-voice.txt

# 验证
cat .claude/tts-voice.txt
# 预期："fr_FR-tom-medium"
```

### 步骤 4.3：测试音频生成

```bash
# 手动测试 TTS 管道
~/.claude/hooks/play-tts.sh "Ceci est un test audio"
```

**预期效果**：以法语男声播放音频。

**排错**：若无声音，请参阅[排错指南](./troubleshooting.md#no-audio)。

### 检查点 4：配置完成 ✅

```bash
# 验证配置文件
test -f .claude/tts-provider.txt && \
test -f .claude/tts-voice.txt && \
test -f .claude/hooks/play-tts.sh && \
echo "✅ Configuration complete"
```

---

## 阶段 5：在 Claude Code 中验证

### 步骤 5.1：启动 Claude Code

```bash
# 启动 Claude Code 会话
claude
```

### 步骤 5.2：测试 TTS 命令

```bash
# 在 Claude 中运行：
/agent-vibes:whoami
# 预期：显示当前语音和提供商

/agent-vibes:list
# 预期：列出所有 15 个以上语音

# 用简单请求测试 TTS
> "Dis-moi bonjour en français"
# 预期：以法语播放音频回复
```

### 步骤 5.3：验证钩子已激活

```bash
# 退出 Claude，检查钩子是否已触发
ls -la /tmp/tts-*.wav
# 预期：临时音频文件

# 检查最后播放的音频
ls -la ~/.claude/tts-last-played.wav
# 预期：文件存在
```

### 检查点 5：验证完成 ✅

```bash
# 最终验证
cat .claude/tts-provider.txt && \
cat .claude/tts-voice.txt && \
piper --help > /dev/null 2>&1 && \
ls ~/.claude/piper-voices/*.onnx | wc -l && \
echo "✅ Installation successful!"
```

---

## 安装后配置

### 降低详细程度（推荐）

```bash
# 在 Claude Code 中执行
/agent-vibes:verbosity low
```

**原因**：减少音频提示频率，降低干扰。

### 隐藏 34 个命令（可选）

```bash
# 在 Claude Code 中执行
/agent-vibes:hide
```

**原因**：整理命令面板。使用 `/agent-vibes:show` 可重新显示。

### 关闭背景音乐

```bash
# 在 Claude Code 中执行
/agent-vibes:background-music off
```

**原因**：背景音乐在专注工作时可能造成干扰。

### 设置项目静音（可选）

```bash
# 仅对当前项目静音 TTS
touch .claude/agentvibes-muted
```

**原因**：某些项目不需要音频（例如纯文档仓库）。

---

## 性能基准测试

**测试系统**：M1 MacBook Pro，16GB RAM，macOS Sequoia 24.6.0

| 指标 | Piper Medium | Piper High | macOS Say |
|------|-------------|------------|-----------|
| **音频生成** | ~200ms | ~400ms | 即时 |
| **总延迟** | ~280ms | ~480ms | ~50ms |
| **内存占用** | ~50MB | ~70MB | ~10MB |
| **CPU 峰值** | 80%（200ms） | 90%（400ms） | 5%（50ms） |
| **语音质量** | ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️ |
| **离线支持** | ✅ | ✅ | ✅ |

**推荐**：Piper Medium = 质量与速度的最佳平衡。

---

## 磁盘占用

| 组件 | 大小 | 位置 |
|------|------|------|
| **Piper TTS** | ~5MB | `~/.local/pipx/venvs/piper-tts/` |
| **语音模型** | ~1GB | `~/.claude/piper-voices/`（15 个语音 × 60MB） |
| **背景音乐** | ~300MB | `.claude/audio/tracks/`（16 首曲目） |
| **脚本** | ~2MB | `.claude/hooks/`（40 个 Bash 脚本） |
| **合计** | **~1.3GB** | - |

**清理建议**：删除不用的语音以节省空间。

---

## 卸载说明

### 自动卸载

```bash
# 完全卸载 Agent Vibes
npx agentvibes uninstall --yes
```

### 手动清理

```bash
# 删除 Agent Vibes 文件
rm -rf .claude/hooks/*vibes*
rm -rf .claude/commands/agent-vibes/
rm -rf .claude/audio/

# 卸载 Piper TTS
pipx uninstall piper-tts

# 删除语音模型
rm -rf ~/.claude/piper-voices/

# 删除配置文件
rm .claude/tts-provider.txt
rm .claude/tts-voice.txt
rm .claude/agentvibes-muted 2>/dev/null
rm ~/.agentvibes-muted 2>/dev/null
```

---

## 常见安装问题

### 问题 1：找不到 `libespeak-ng.1.dylib`

**症状**：
```
dyld[xxx]: Library not loaded: @rpath/libespeak-ng.1.dylib
```

**解决方案**：
```bash
# 安装 espeak-ng
brew install espeak-ng

# 通过 pipx 重新安装 Piper（而非二进制版本）
pipx uninstall piper-tts
pipx install piper-tts
```

### 问题 2：`flock` 警告（可选工具）

**症状**：
```
⚠ flock - TTS queue locking
```

**影响**：轻微。TTS 不依赖 `flock` 也能工作，但在消息快速发送时可能出现音频冲突。

**解决方案**（可选）：
```bash
# flock 已包含在 util-linux 中
# 若需要，将其加入 PATH
export PATH="/opt/homebrew/opt/util-linux/bin:$PATH"
```

### 问题 3：Agent Vibes 安装器意外退出

**症状**：
```
ExitPromptError: User force closed the prompt
```

**原因**：交互式安装器需要用户手动操作，无法自动化。

**解决方案**：在交互式终端中运行 `npx agentvibes install`（不要通过脚本调用）。

### 问题 4：安装后无音频

**诊断步骤**：
```bash
# 1. 检查静音状态
ls .claude/agentvibes-muted ~/.agentvibes-muted

# 2. 检查提供商
cat .claude/tts-provider.txt

# 3. 手动测试 Piper
echo "Test" | piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test.wav && afplay /tmp/test.wav
```

**解决方案**：参阅[排错指南](./troubleshooting.md)。

---

## 后续步骤

安装完成后：

1. **[语音目录](./voice-catalog.md)** - 探索 15 种语音，选择你喜欢的
2. **[README](./README.md)** - 了解常用命令和使用场景
3. **[排错指南](./troubleshooting.md)** - 解决常见问题
4. **[AI 生态系统](../../../guide/ecosystem/ai-ecosystem.md#47-voice-interfaces)** - TTS 在更广泛 AI 生态中的位置

---

## 资源链接

- **Agent Vibes GitHub**：https://github.com/paulpreibisch/AgentVibes
- **Piper 语音仓库**：https://huggingface.co/rhasspy/piper-voices
- **Piper 语音试听**：https://rhasspy.github.io/piper-samples/
- **Agent Vibes 官网**：https://agentvibes.org

---

*安装指南由 [Claude Code Ultimate Guide](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide) 维护*
*最后更新：2026-01-22 | 测试环境：macOS Sequoia 24.6.0（Apple Silicon）*
