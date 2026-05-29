> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: talk-stage4-position
description: "生成 3-4 个战略性演讲角度，包含优缺点分析、标题选项、CFP 描述和同行反馈草稿，然后强制执行检查点等待用户确认，之后才进入脚本阶段。适用于决定演讲框架、准备 CFP 投稿或在多个叙事角度之间做选择时使用。"
tags: [talk, pipeline, presentation, stage-4, checkpoint]
allowed-tools: "Write, Read, AskUserQuestion"
effort: high
---

# 演讲第 4 阶段：定位 + 检查点

生成战略性角度、标题、描述和同行反馈草稿，然后**停止并等待**你确认角度和标题选择，第 5 阶段才能继续。

## 适用场景

- 第 3 阶段（概念）之后——需要概念目录
- 决定演讲框架时
- 投递 CFP 之前（直接使用生成的描述）

## 此技能的功能

1. **读取输入** — 摘要 + 概念 + 活动约束条件
2. **生成角度** — 3-4 个不同角度，含优缺点分析
3. **给出推荐** — 明确的选择建议及结构化理由
4. **生成标题** — 每个角度 3-5 个选项
5. **生成描述** — 简短摘要 + 完整 CFP 描述
6. **生成反馈草稿** — 可直接发送的消息（3 种格式）
7. **检查点** — 显示选择请求并等待用户回应
8. **保存 4 个文件**

## 输入

- `talks/{YYYY}-{slug}-summary.md`（必填）
- `talks/{YYYY}-{slug}-concepts.md`（必填）
- 活动约束：时长、受众、CFP 格式（如适用）

## 输出

- `talks/{YYYY}-{slug}-angles.md`
- `talks/{YYYY}-{slug}-titre.md`
- `talks/{YYYY}-{slug}-descriptions.md`
- `talks/{YYYY}-{slug}-feedback-draft.md`

## angles.md 格式

```markdown
# Talk Angles — {provisional title}

**Goal**: Choose the angle that maximizes impact for {audience}.
**Audience**: {audience description}

---

## Angle 1: {Angle name}

**Pitch**: {2-3 sentences describing the talk from this angle}

**Strengths**:
- {strength 1}
- {strength 2}

**Weaknesses**:
- {weakness 1}
- {weakness 2}

**Audience fit**: Strong / Medium / Weak — {short justification}

**Verdict**: ⭐⭐⭐⭐⭐ (out of 5)

---

[Angle 2, Angle 3, (optional Angle 4) — same structure]

---

## Recommendation: Angle {X}, enriched by the others

**Angle {X} is the right choice.** Here's why:

### 1. It's the only angle that integrates the others
[Structure showing how other angles feed into the main one]

### 2. The narrative arc is natural and compelling
[Why the story holds better with this angle]

### 3. The metrics lend credibility throughout
[Which metrics support this angle most]

### 4. The final message emerges naturally
[How the conclusion flows from this angle]

---

## Recommended structure with sub-angles

| Act | Duration | Main angle | Integrated sub-angle |
|-----|----------|-----------|---------------------|
| 1. {name} | {n} min | {main angle} | {sub-angle} |
...
```

## titre.md 格式

```markdown
# Titles — Talk {slug}

**Selected angle**: Angle {X} — {name}
**Constraints**: {duration} min | {audience}

---

## Titles for the recommended angle

### Option 1 (recommended)
**{Main title}**
*Optional subtitle: {subtitle}*

Strengths: {why this title works}
Audience appeal: {who it hooks}

### Option 2
**{Title}**
Strengths: {strengths}

[Options 3-5]

---

## Titles for alternative angles (backup)

### If Angle 2 chosen
- **{title}**
- **{title}**

[If Angle 3 chosen — same]

---

## Verdict

**Recommendation**: Option 1 — "{title}"
**Why**: {short justification}
```

## descriptions.md 格式

```markdown
# Descriptions — Talk {slug}

---

## Short description (abstract, ~100 words)

{Full text — direct, engaging, starts with the impact or concrete promise.
Not "In this talk, we will..."}

---

## Long description (CFP, ~250 words)

{Full text — context, what the audience will learn, who it's for.
Includes key metrics if available.
Direct and factual tone.}

---

## Speaker pitch (bio-ready, ~50 words)

{Speaker introduction in 1-2 sentences, their relationship to the topic}

---

## Tags / Keywords

{5-10 relevant tags for CFP or search}
```

## 检查点（强制执行 — 第 7 步）

生成并保存 4 个文件后，显示：

```
---
CHECKPOINT: Angle + Title choice

I've generated 4 files:
- talks/{YYYY}-{slug}-angles.md    → {n} angles analyzed
- talks/{YYYY}-{slug}-titre.md     → {n} title options
- talks/{YYYY}-{slug}-descriptions.md
- talks/{YYYY}-{slug}-feedback-draft.md

Before starting the script (Stage 5), I need your choice:

1. Which angle do you choose? (recommended: Angle {X} — {name})
2. Which title do you prefer? (recommended: "{title}")

You can also modify, combine, or propose something different.
Reply to start the script.
---
```

**未经用户明确确认，不得调用第 5 阶段。**

## 角度生成规则

- 最少 3 个角度，最多 4 个（超过则形成噪声）
- 每个角度必须真正不同（不能是同一角度的变体）
- 推荐必须明确且有论据——不能是"你来选"
- 始终测试："这个角度能在不重复的情况下撑满整个时长吗？"

## 反模式

- 标题党（"关于 AI 没人告诉你的事"）
- 默认推荐最后列出的角度（近因偏差）
- 描述读起来像幻灯片摘要
- 跳过检查点——这是流水线中最重要的控制节点
- 描述中使用营销语言（革命性的、颠覆性的）

## 验证清单

- [ ] 3-4 个角度，含优缺点和受众适配分析
- [ ] 明确的推荐及结构化理由
- [ ] 推荐角度的 3-5 个标题
- [ ] 简短描述（约 100 字）和完整描述（约 250 字）
- [ ] 从模板生成反馈草稿
- [ ] 检查点清晰显示
- [ ] 4 个文件已保存

## 技巧

- 在检查点之前将 `feedback-draft.md` 发给同行——10 分钟的外部反馈可以节省数小时的脚本返工
- 推荐是起点，不是命令——你对受众的了解优先于任何算法建议
- 弱标题通常过于抽象：用这个问题测试每个标题——"走廊里的人看到这个标题会停下脚步吗？"

## 模板

- 同行反馈格式：[`templates/feedback-draft.md`](templates/feedback-draft.md)

## 相关内容

- [第 3 阶段：概念](../stage-3-concepts/SKILL.md) — 前置条件
- [第 5 阶段：脚本](../stage-5-script/SKILL.md) — 通过此检查点后启动
- [编排器](../orchestrator/SKILL.md)
