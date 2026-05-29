> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Agent Vibes - 完整语音目录"
description: "可用 TTS 语音的参考目录，含质量评级和语言支持说明"
tags: [reference, tts, integration]
---

# Agent Vibes - 完整语音目录

**语音总数**：15 个已安装 + 50+ 可用
**支持语言**：法语（4 个模型，128 个说话人）、英语（12 个模型）

---

## 法语语音（Voix Françaises）

### 概览

| 语音 ID | 性别 | 质量 | 说话人数 | 大小 | 推荐度 |
|---------|------|------|----------|------|--------|
| **fr_FR-tom-medium** | 男 | 中等 | 1 | 60MB | ⭐️⭐️⭐️⭐️⭐️ 最佳法语男声 |
| fr_FR-siwis-medium | 女 | 中等 | 1 | 60MB | ⭐️⭐️⭐️⭐️ 清晰自然 |
| fr_FR-upmc-medium | 中性 | 中等 | 1 | 73MB | ⭐️⭐️⭐️ 多用途 |
| **fr_FR-mls-medium** | 混合 | 中等 | **124** | 73MB | ⭐️⭐️⭐️⭐️⭐️ 最多样化 |

---

### fr_FR-tom-medium ⭐️⭐️⭐️⭐️⭐️

**性别**：男
**质量**：中等
**特点**：专业，发音清晰，中性口音
**适用场景**：技术文档、代码审查、专业场合
**延迟**：约 200ms

**用法**：
```bash
# 在 Claude Code 中
/agent-vibes:switch fr_FR-tom-medium

# 手动测试
echo "Bonjour, je m'appelle Tom. Je suis une voix synthétique française." | \
  piper -m ~/.claude/piper-voices/fr_FR-tom-medium.onnx \
  --output-file /tmp/tom.wav && afplay /tmp/tom.wav
```

**音频示例**：在 https://rhasspy.github.io/piper-samples/ 收听（搜索 "fr_FR tom"）

---

### fr_FR-siwis-medium ⭐️⭐️⭐️⭐️

**性别**：女
**质量**：中等
**特点**：温暖自然，略带瑞士口音
**适用场景**：教程、教育内容、友好互动
**延迟**：约 200ms

**用法**：
```bash
# 在 Claude Code 中
/agent-vibes:switch fr_FR-siwis-medium

# 手动测试
echo "Bonjour, je suis Siwis. J'ai une voix claire et agréable." | \
  piper -m ~/.claude/piper-voices/fr_FR-siwis-medium.onnx \
  --output-file /tmp/siwis.wav && afplay /tmp/siwis.wav
```

**音频示例**：https://rhasspy.github.io/piper-samples/（搜索 "fr_FR siwis"）

---

### fr_FR-upmc-medium ⭐️⭐️⭐️

**性别**：中性（略偏女性）
**质量**：中等
**特点**：技术性强，精准，学术语调
**适用场景**：科学内容、数据分析、正式报告
**延迟**：约 220ms（模型较大）

**用法**：
```bash
# 在 Claude Code 中
/agent-vibes:switch fr_FR-upmc-medium

# 手动测试
echo "Analyse des données en cours. Résultats disponibles dans quelques instants." | \
  piper -m ~/.claude/piper-voices/fr_FR-upmc-medium.onnx \
  --output-file /tmp/upmc.wav && afplay /tmp/upmc.wav
```

**音频示例**：https://rhasspy.github.io/piper-samples/（搜索 "fr_FR upmc"）

---

### fr_FR-mls-medium ⭐️⭐️⭐️⭐️⭐️（多说话人）

**性别**：混合（49 位女性，75 位男性）
**质量**：中等
**说话人数**：**124 种不同声音**
**特点**：多样化（年轻、年老、口音、语调各异）
**适用场景**：对话模拟、多样性需求、角色配音
**延迟**：约 200ms + 说话人选择时间

**用法**：
```bash
# 在 Claude Code 中（使用默认说话人）
/agent-vibes:switch fr_FR-mls-medium

# 手动测试并指定说话人（0-123）
echo "Je suis le speaker numéro 42" | \
  piper -m ~/.claude/piper-voices/fr_FR-mls-medium.onnx -s 42 \
  --output-file /tmp/mls-42.wav && afplay /tmp/mls-42.wav

# 测试多个说话人
for speaker in {0..5}; do
  echo "Bonjour, speaker $speaker" | \
    piper -m ~/.claude/piper-voices/fr_FR-mls-medium.onnx -s $speaker \
    --output-file /tmp/mls-$speaker.wav
  afplay /tmp/mls-$speaker.wav
  sleep 1
done
```

**说话人范围**：
- `0-48`：女声（共 49 位）
- `49-123`：男声（共 75 位）

**推荐说话人**：
| 说话人 ID | 性别 | 特点 |
|-----------|------|------|
| 7 | 女 | 年轻，有活力 |
| 15 | 女 | 成熟，专业 |
| 23 | 女 | 温暖，友好 |
| 55 | 男 | 低沉，权威 |
| 72 | 男 | 清晰，技术性 |
| 99 | 男 | 年轻，随性 |

**音频示例**：https://rhasspy.github.io/piper-samples/（搜索 "fr_FR mls"）

---

## 英语语音（Voix Anglaises）

### 概览

| 语音 ID | 性别 | 质量 | 特点 | 推荐度 |
|---------|------|------|------|--------|
| **en_US-ryan-high** | 男 | 高 | 专业 | ⭐️⭐️⭐️⭐️⭐️ 最佳英语男声 |
| en_US-amy-medium | 女 | 中等 | 温暖自然 | ⭐️⭐️⭐️⭐️ |
| en_US-lessac-medium | 男 | 中等 | 权威 | ⭐️⭐️⭐️⭐️ |
| en_US-libritts-high | 混合 | 高 | 非常自然 | ⭐️⭐️⭐️⭐️⭐️ |
| en_US-hfc_female-medium | 女 | 中等 | 技术性 | ⭐️⭐️⭐️ |
| en_US-bryce-medium | 男 | 中等 | 年轻，动感 | ⭐️⭐️⭐️ |
| en_US-danny-low | 男 | 低 | 快速，高效 | ⭐️⭐️ |
| en_US-kathleen-low | 女 | 低 | 快速，高效 | ⭐️⭐️ |
| en_US-kusal-medium | 男 | 中等 | 印度口音 | ⭐️⭐️⭐️ |
| en_US-kristin-medium | 女 | 中等 | 清晰，中性 | ⭐️⭐️⭐️⭐️ |
| en_US-libritts_r-high | 混合 | 高 | 非常自然 | ⭐️⭐️⭐️⭐️⭐️ |
| 16Speakers | 多人 | 中等 | 16 种声音 | ⭐️⭐️⭐️⭐️ |

---

### en_US-ryan-high ⭐️⭐️⭐️⭐️⭐️

**性别**：男
**质量**：高（最佳英语语音）
**特点**：专业新闻主播风格，发音清晰
**适用场景**：专业演示、文档、代码审查
**延迟**：约 400ms（高质量模型）

**用法**：
```bash
/agent-vibes:switch en_US-ryan-high
```

---

### en_US-amy-medium ⭐️⭐️⭐️⭐️

**性别**：女
**质量**：中等
**特点**：温暖，友好，对话感强
**适用场景**：教程、日常互动、教育内容
**延迟**：约 200ms

**用法**：
```bash
/agent-vibes:switch en_US-amy-medium
```

---

### en_US-libritts-high ⭐️⭐️⭐️⭐️⭐️

**性别**：混合（多说话人）
**质量**：高
**特点**：非常自然，富有表现力
**适用场景**：高质量音频旁白
**延迟**：约 400ms

**用法**：
```bash
/agent-vibes:switch en_US-libritts-high
```

---

### 16Speakers ⭐️⭐️⭐️⭐️（英语多说话人）

**性别**：混合（8 位女性，8 位男性）
**质量**：中等
**说话人数**：16 种不同英语声音
**特点**：年龄、口音、语调各有差异
**适用场景**：对话、多样化需求、角色旁白
**延迟**：约 200ms + 说话人选择时间

**用法**：
```bash
# 默认说话人
/agent-vibes:switch 16Speakers

# 指定说话人（0-15）
echo "I am speaker number 5" | \
  piper -m ~/.claude/piper-voices/16Speakers.onnx -s 5 \
  --output-file /tmp/16sp-5.wav && afplay /tmp/16sp-5.wav
```

---

## 低质量语音（速度更快，质量较低）

### 何时使用低质量语音

- 节省电量（生成速度提升 50%）
- 对延迟敏感的应用（要求 <150ms）
- 快速原型或测试
- 后台通知（质量要求不高）

### 可用的低质量模型

| 语音 ID | 性别 | 延迟 | 质量 |
|---------|------|------|------|
| en_US-danny-low | 男 | ~100ms | ⭐️⭐️ |
| en_US-kathleen-low | 女 | ~100ms | ⭐️⭐️ |
| fr_FR-gilles-low | 男 | ~100ms | ⭐️⭐️ |
| fr_FR-siwis-low | 女 | ~100ms | ⭐️⭐️ |

**下载低质量语音**：
```bash
cd ~/.claude/piper-voices
curl -L -o fr_FR-gilles-low.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/gilles/low/fr_FR-gilles-low.onnx"
curl -L -o fr_FR-gilles-low.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/gilles/low/fr_FR-gilles-low.onnx.json"
```

---

## 高质量语音（速度较慢，质量更好）

### 何时使用高质量语音

- 专业演示
- 内容创作（视频、播客）
- 演示或公开展示
- 对延迟要求不高的场景

### 可用的高质量模型

| 语音 ID | 性别 | 延迟 | 质量 |
|---------|------|------|------|
| en_US-ryan-high | 男 | ~400ms | ⭐️⭐️⭐️⭐️⭐️ |
| en_US-libritts-high | 混合 | ~400ms | ⭐️⭐️⭐️⭐️⭐️ |
| en_US-libritts_r-high | 混合 | ~400ms | ⭐️⭐️⭐️⭐️⭐️ |
| fr_FR-siwis-high | 女 | ~400ms | ⭐️⭐️⭐️⭐️⭐️ |

**下载高质量语音**：
```bash
cd ~/.claude/piper-voices
curl -L -o fr_FR-siwis-high.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/high/fr_FR-siwis-high.onnx"
curl -L -o fr_FR-siwis-high.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/fr/fr_FR/siwis/high/fr_FR-siwis-high.onnx.json"
```

---

## 其他语言

Piper TTS 支持 **50+ 种语言**。可从 Hugging Face 下载更多语音。

### 常用语言

| 语言 | 可用语音数 | 仓库链接 |
|------|-----------|----------|
| 西班牙语（es_ES） | 10+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/es/es_ES) |
| 德语（de_DE） | 8+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/de/de_DE) |
| 意大利语（it_IT） | 6+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/it/it_IT) |
| 葡萄牙语（pt_BR） | 5+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/pt/pt_BR) |
| 俄语（ru_RU） | 4+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/ru/ru_RU) |
| 中文（zh_CN） | 3+ | [链接](https://huggingface.co/rhasspy/piper-voices/tree/main/zh/zh_CN) |

### 下载西班牙语语音示例

```bash
cd ~/.claude/piper-voices

# 西班牙语男声（Davefx - 高质量）
curl -L -o es_ES-davefx-medium.onnx \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/es/es_ES/davefx/medium/es_ES-davefx-medium.onnx"
curl -L -o es_ES-davefx-medium.onnx.json \
  "https://huggingface.co/rhasspy/piper-voices/resolve/main/es/es_ES/davefx/medium/es_ES-davefx-medium.onnx.json"

# 测试
echo "Hola, soy Claude y hablo español" | \
  piper -m es_ES-davefx-medium.onnx \
  --output-file /tmp/es.wav && afplay /tmp/es.wav
```

---

## 语音选择推荐

### 按使用场景

| 使用场景 | 推荐语音 | 理由 |
|----------|----------|------|
| **技术文档** | fr_FR-tom-medium | 清晰、专业、技术语调 |
| **代码审查** | en_US-ryan-high | 权威，发音清晰 |
| **教程** | fr_FR-siwis-medium | 温暖、友好、适合教学 |
| **后台通知** | fr_FR-gilles-low | 快速、高效、低延迟 |
| **专业演示** | en_US-ryan-high | 最高质量，专业感强 |
| **多样化/对话** | fr_FR-mls-medium | 124 种不同声音 |
| **省电优化** | 任意 "-low" 语音 | 生成速度提升 50% |

### 按语言

| 主要语言 | 最佳语音 | 备选语音 |
|----------|----------|----------|
| **法语** | fr_FR-tom-medium | fr_FR-mls-medium（多样化） |
| **英语** | en_US-ryan-high | en_US-libritts-high |
| **西班牙语** | es_ES-davefx-medium | es_ES-carlfm-medium |
| **德语** | de_DE-thorsten-high | de_DE-kerstin-low |

---

## 语音对比工具

并排对比多个语音：

```bash
# 创建对比脚本
cat > /tmp/compare-voices.sh << 'EOF'
#!/bin/bash
TEXT="$1"
VOICES=("fr_FR-tom-medium" "fr_FR-siwis-medium" "fr_FR-upmc-medium")

for voice in "${VOICES[@]}"; do
  echo "Testing $voice..."
  echo "$TEXT" | piper -m ~/.claude/piper-voices/${voice}.onnx \
    --output-file /tmp/${voice}.wav
  afplay /tmp/${voice}.wav
  sleep 2
done
EOF

chmod +x /tmp/compare-voices.sh

# 对比语音
/tmp/compare-voices.sh "Bonjour, ceci est un test de comparaison"
```

---

## 资源

- **Piper 语音仓库**：https://huggingface.co/rhasspy/piper-voices
- **音频示例**：https://rhasspy.github.io/piper-samples/
- **语音训练**：https://github.com/rhasspy/piper/blob/master/TRAINING.md
- **自定义语音**：https://community.rhasspy.org/c/piper/

---

*语音目录由 [Claude Code Ultimate Guide](https://github.com/claude-code-ultimate-guide/claude-code-ultimate-guide) 维护*
*最后更新：2026-01-22 | Piper TTS v1.3.0*
