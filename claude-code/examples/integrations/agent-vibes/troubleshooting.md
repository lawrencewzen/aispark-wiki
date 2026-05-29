> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Agent Vibes - 故障排查指南"
description: "Agent Vibes TTS 常见问题的诊断步骤与解决方案"
tags: [guide, tts, debugging, integration]
---

# Agent Vibes - 故障排查指南

**常见问题**：7 种场景及逐步解决方案
**诊断工具**：用于问题识别的命令与脚本

---

## 问题 1：无音频输出

### 现象
```
Claude 有响应但没有播放 TTS 音频
```

### 诊断步骤

```bash
# 1. 检查是否已静音
ls -la .claude/agentvibes-muted ~/.agentvibes-muted
# 若文件存在 → TTS 已静音

# 2. 检查 provider 配置
cat .claude/tts-provider.txt
# 预期值："piper" 或 "macos"

# 3. 检查语音配置
cat .claude/tts-voice.txt
# 预期值：语音名称，如 "fr_FR-tom-medium"

# 4. 直接测试 Piper
echo "Test audio" | piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test.wav && afplay /tmp/test.wav
```

### 解决方案

**如果已静音**：
```bash
# 取消静音
rm .claude/agentvibes-muted
rm ~/.agentvibes-muted 2>/dev/null

# 或在 Claude Code 中执行
/agent-vibes:unmute
```

**如果 provider 配置有误**：
```bash
# 将 provider 设置为 Piper
echo "piper" > .claude/tts-provider.txt

# 验证
cat .claude/tts-provider.txt
```

**如果缺少语音文件**：
```bash
# 下载缺失的语音
cd ~/.claude/piper-voices
curl -L -o fr_FR-tom-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx"
curl -L -o fr_FR-tom-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/tom/medium/fr_FR-tom-medium.onnx.json"
```

**如果 Piper 命令失败**：
```bash
# 通过 pipx 重新安装 Piper
pipx uninstall piper-tts
pipx install piper-tts

# 验证
piper --help
```

---

## 问题 2：libespeak-ng.1.dylib 未找到

### 现象
```
dyld[xxx]: Library not loaded: @rpath/libespeak-ng.1.dylib
  Referenced from: /Users/.../piper
  Reason: tried: '/usr/local/lib/libespeak-ng.1.dylib' (no such file)
```

### 根本原因
Piper 二进制文件依赖 `espeak-ng` 库，但该库尚未安装。

### 解决方案

```bash
# 安装 espeak-ng
brew install espeak-ng

# 验证安装
espeak-ng --version
# 预期：eSpeak NG text-to-speech: 1.52.0

# 查找库文件位置
find /opt/homebrew -name "libespeak-ng*.dylib"
# 预期：/opt/homebrew/lib/libespeak-ng.1.dylib

# 若 Piper 仍然失败，通过 pipx 重新安装（而非使用二进制）
pipx uninstall piper-tts 2>/dev/null
pipx install piper-tts

# 再次测试
piper --help
```

### 预防措施
务必在安装 Piper TTS **之前**先安装 `espeak-ng`。

---

## 问题 3：语音听起来机械或质量低

### 现象
```
音频可以播放，但语音质量差、机械感强、不自然
```

### 诊断

```bash
# 检查加载的语音模型
cat .claude/tts-voice.txt

# 检查语音质量级别
ls -lh ~/.claude/piper-voices/*.onnx | grep $(cat .claude/tts-voice.txt)
# 低质量：~20-30MB
# 中质量：~50-70MB
# 高质量：~100-150MB
```

### 解决方案

**升级为高质量模型**：
```bash
# 下载高质量法语语音
cd ~/.claude/piper-voices
curl -L -o fr_FR-siwis-high.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/high/fr_FR-siwis-high.onnx"
curl -L -o fr_FR-siwis-high.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/high/fr_FR-siwis-high.onnx.json"

# 切换到高质量语音
/agent-vibes:switch fr_FR-siwis-high
```

**备选方案：尝试不同语音**：
```bash
# 预览多个语音，找到偏好的
/agent-vibes:preview

# 或手动测试
for voice in fr_FR-tom-medium fr_FR-siwis-medium fr_FR-upmc-medium; do
  echo "Test voix $voice" | \
    piper -m ~/.claude/piper-voices/${voice}.onnx \
    --output-file /tmp/${voice}.wav
  afplay /tmp/${voice}.wav
  sleep 2
done
```

**质量对比**：
| 质量 | 大小 | 延迟 | 自然度 |
|------|------|------|--------|
| 低 | ~25MB | ~100ms | ⭐️⭐️ |
| 中 | ~60MB | ~200ms | ⭐️⭐️⭐️⭐️ |
| 高 | ~120MB | ~400ms | ⭐️⭐️⭐️⭐️⭐️ |

---

## 问题 4：高延迟（>500ms）

### 现象
```
Claude 响应与音频播放之间有明显延迟
```

### 诊断

```bash
# 计时音频生成速度
time (echo "Test rapide" | \
  piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/test.wav > /dev/null 2>&1)
# 预期：中质量约 0.2s

# 检查音频效果是否已启用
cat .claude/config/audio-effects.cfg 2>/dev/null
# 查找 REVERB_ENABLED=true、ECHO_ENABLED=true

# 检查背景音乐是否已启用
cat .claude/config/background-music.cfg 2>/dev/null
# 查找 ENABLED=true
```

### 解决方案

**切换到低质量语音**（速度提升 50%）：
```bash
cd ~/.claude/piper-voices
curl -L -o fr_FR-gilles-low.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/gilles/low/fr_FR-gilles-low.onnx"
curl -L -o fr_FR-gilles-low.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/gilles/low/fr_FR-gilles-low.onnx.json"

/agent-vibes:switch fr_FR-gilles-low
```

**禁用音频效果**：
```bash
# 在 Claude Code 中
/agent-vibes:effects off

# 或手动配置
echo "REVERB_ENABLED=false" > .claude/config/audio-effects.cfg
echo "ECHO_ENABLED=false" >> .claude/config/audio-effects.cfg
```

**禁用背景音乐**：
```bash
# 在 Claude Code 中
/agent-vibes:background-music off

# 或手动配置
echo "ENABLED=false" > .claude/config/background-music.cfg
```

**切换到 macOS Say**（即时响应，质量较低）：
```bash
/agent-vibes:provider switch macos
```

**性能对比**：
| 配置 | 延迟 | 质量 |
|------|------|------|
| Piper 高质量 + 效果 + 音乐 | ~500ms | ⭐️⭐️⭐️⭐️⭐️ |
| Piper 中质量 + 效果 | ~280ms | ⭐️⭐️⭐️⭐️ |
| Piper 中质量（无效果） | ~200ms | ⭐️⭐️⭐️⭐️ |
| Piper 低质量（无效果） | ~100ms | ⭐️⭐️ |
| macOS Say | ~50ms | ⭐️⭐️⭐️ |

---

## 问题 5：Agent Vibes 命令占满命令面板

### 现象
```
34 个 /agent-vibes:* 命令占满了 Claude Code 命令面板
```

### 解决方案

```bash
# 隐藏所有 Agent Vibes 命令
/agent-vibes:hide

# 命令仍可正常使用，只是从自动补全中隐藏

# 需要时重新显示
/agent-vibes:show
```

### 备选方案：完全卸载 Agent Vibes

```bash
npx agentvibes uninstall --yes
```

---

## 问题 6：音频重复播放（回声/重复）

### 现象
```
同一段音频在短时间内播放 2-3 次
```

### 根本原因
`flock`（文件锁）不可用，导致快速消息之间产生竞态条件。

### 诊断

```bash
# 检查 flock 是否可用
which flock || /opt/homebrew/opt/util-linux/bin/flock --version
```

### 解决方案

```bash
# 安装 util-linux（包含 flock）
brew install util-linux

# 将 flock 加入 PATH（可选）
echo 'export PATH="/opt/homebrew/opt/util-linux/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 验证
flock --version
```

### 临时替代方案（flock 不可用时）
```bash
# 降低详细程度以减少快速消息
/agent-vibes:verbosity low
```

---

## 问题 7：安装卡住或意外退出

### 现象
```
npx agentvibes install 退出并显示：
ExitPromptError: User force closed the prompt
```

### 根本原因
交互式安装程序需要终端输入，无法自动化执行。

### 解决方案

```bash
# 确保在交互式终端中运行（而非脚本中）
# 直接在终端运行，不要通过自动化工具调用

npx agentvibes install

# 如果仍然失败，尝试不使用缓存
npx --yes agentvibes@latest install
```

### 备选方案：手动安装

```bash
# 若安装程序多次失败，手动安装各组件

# 1. 安装 Piper TTS
pipx install piper-tts

# 2. 下载语音文件
mkdir -p ~/.claude/piper-voices
cd ~/.claude/piper-voices
# ... 手动下载语音（参见 voice-catalog.md）

# 3. 创建配置文件
echo "piper" > .claude/tts-provider.txt
echo "fr_FR-tom-medium" > .claude/tts-voice.txt

# 4. 下载 Agent Vibes 脚本
# （联系维护者或从 npm 包中提取）
```

---

## 诊断脚本

创建综合诊断脚本：

```bash
cat > /tmp/agent-vibes-diagnostic.sh << 'EOF'
#!/bin/bash
echo "=== Agent Vibes Diagnostic ==="
echo ""

echo "1. System Dependencies"
command -v /opt/homebrew/bin/bash && echo "  ✓ Bash 5.x" || echo "  ✗ Bash 5.x missing"
command -v sox && echo "  ✓ sox" || echo "  ✗ sox missing"
command -v ffmpeg && echo "  ✓ ffmpeg" || echo "  ✗ ffmpeg missing"
command -v espeak-ng && echo "  ✓ espeak-ng" || echo "  ✗ espeak-ng missing"
command -v piper && echo "  ✓ piper" || echo "  ✗ piper missing"
echo ""

echo "2. Configuration Files"
test -f .claude/tts-provider.txt && echo "  ✓ Provider: $(cat .claude/tts-provider.txt)" || echo "  ✗ Provider not configured"
test -f .claude/tts-voice.txt && echo "  ✓ Voice: $(cat .claude/tts-voice.txt)" || echo "  ✗ Voice not configured"
test -f .claude/hooks/play-tts.sh && echo "  ✓ TTS hook installed" || echo "  ✗ TTS hook missing"
echo ""

echo "3. Mute Status"
test -f .claude/agentvibes-muted && echo "  ⚠ Project muted" || echo "  ✓ Project unmuted"
test -f ~/.agentvibes-muted && echo "  ⚠ Global muted" || echo "  ✓ Global unmuted"
echo ""

echo "4. Voice Models"
echo "  Installed voices: $(ls ~/.claude/piper-voices/*.onnx 2>/dev/null | wc -l)"
ls ~/.claude/piper-voices/*.onnx 2>/dev/null | sed 's|.*/||' | sed 's|.onnx||' | sed 's/^/    - /'
echo ""

echo "5. Audio Test"
if command -v piper > /dev/null 2>&1; then
  echo "Test" | piper -m ~/.claude/piper-voices/$(cat .claude/tts-voice.txt).onnx \
    --output-file /tmp/diagnostic-test.wav 2>&1 | grep -q "error" && \
    echo "  ✗ Audio generation failed" || echo "  ✓ Audio generation successful"
  afplay /tmp/diagnostic-test.wav 2>/dev/null && echo "  ✓ Audio playback successful" || echo "  ✗ Audio playback failed"
else
  echo "  ✗ Piper not installed, skipping test"
fi
echo ""

echo "=== End Diagnostic ==="
EOF

chmod +x /tmp/agent-vibes-diagnostic.sh

# 运行诊断
/tmp/agent-vibes-diagnostic.sh
```

---

## 获取帮助

如果问题仍然存在：

1. **运行诊断脚本**（见上文）并分享输出结果
2. **查看 Agent Vibes GitHub Issues**：https://github.com/paulpreibisch/AgentVibes/issues
3. **查看 Claude Code 指南**：[AI 生态系统](../../../guide/ecosystem/ai-ecosystem.md)
4. **查看日志**：`tail -f ~/.claude/tts-debug.log`（如果存在）

---

## 已知限制

| 限制 | 影响 | 替代方案 |
|------|------|----------|
| Bash 3.2（macOS 默认版本） | 脚本运行失败 | 通过 Homebrew 安装 Bash 5.x |
| 不支持 Windows | 无法原生运行 | 使用 WSL2 或 macOS |
| flock 为可选依赖 | 音频可能重叠播放 | 安装 util-linux 或降低详细程度 |
| 语音文件较大 | 约占 1GB 磁盘空间 | 删除未使用的语音 |
| 高质量模式延迟 | 约 400ms 延迟 | 使用中质量或低质量 |

---

*故障排查指南由 [Claude Code Ultimate Guide](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide) 维护*
*最后更新：2026-01-22 | Agent Vibes v3.0.0*
