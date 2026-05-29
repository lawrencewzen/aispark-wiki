> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: talk-stage3-concepts
description: "根据演讲摘要和时间线，构建带编号的分类概念目录，对每个概念的演讲潜力进行 HIGH / MEDIUM / LOW 评分，并可选性地结合代码库进行增强。在选择演讲角度之前需要结构化概念清单时使用，或用于评估哪些想法具有最强的演讲潜力。"
tags: [talk, pipeline, presentation, stage-3]
allowed-tools: "Write, Read"
effort: high
---

# 演讲第3阶段：概念

对源材料中所有可识别的概念构建详尽目录。每个概念均带有编号、分类，并对其演讲潜力进行评分。

## 适用场景

- 在第1阶段之后（如果是 REX 模式则在第2阶段之后）
- 在第4阶段之前（定位阶段需要概念目录）
- 在选择角度之前，需要结构化清单了解可用内容时

## 此技能的功能

1. **读取摘要** — 加载 `{slug}-summary.md`
2. **读取时间线**（如可用）— 用经过验证的日期丰富评分
3. **提取概念** — 对源材料进行全面扫描
4. **分类** — 将每个概念分配到一个领域类别
5. **评分** — 对演讲潜力进行 HIGH / MEDIUM / LOW 评分
6. **可选代码库增强** — 若提供了 repo_path，则分析 AI 配置概念
7. **写入输出文件**

## 输入

- `talks/{YYYY}-{slug}-summary.md`（必填）
- `talks/{YYYY}-{slug}-timeline.md`（可选——丰富 REX 概念）
- `repo_path`（可选——用于提取配置/基础设施概念）

## 输出

- `talks/{YYYY}-{slug}-concepts.md`（主目录）
- `talks/{YYYY}-{slug}-concepts-enriched.md`（若提供了 repo_path）

## 评分标准

### HIGH — 强潜力
- 可现场演示或截图展示
- 反直觉或令人意外（能引发反应）
- 关联可验证的数字
- 具体且可操作（30秒内可解释清楚）
- 相较于同主题其他演讲具有差异化

### MEDIUM — 中等潜力
- 有用但在预期之内（不令人意外）
- 缺少具体证明或数字
- 过于特定于某一具体情境
- 在30分钟演讲中需要过多解释

### LOW — 弱潜力
- 过于抽象或哲学性，缺乏具体落地
- 已被其他演讲者大量覆盖
- 需要特定技术背景
- 难以在幻灯片中呈现

**评分纪律**：HIGH 最多占30%。如果所有概念都是 HIGH，则 HIGH 毫无意义。

## 标准类别

| 类别 | 描述 |
|------|------|
| **Architecture（架构）** | 技术决策、技术栈、结构模式 |
| **Tooling（工具）** | 工具、工作流、自动化 |
| **Philosophy（哲学）** | 原则、思维方式、方法论 |
| **Workflow（工作流）** | 工作流程、习惯 |
| **Knowledge Transfer（知识传递）** | 入职、团队、知识共享 |
| **Problems（问题）** | 遇到的障碍、权衡取舍 |
| **Open Source（开源）** | 贡献、分享、社区 |
| **AI Config（AI 配置）** | AI 配置、配置文件、知识输入 |
| **AI Infrastructure（AI 基础设施）** | 智能体、技能、钩子、命令 |
| **AI Quality（AI 质量）** | 审查、测试、反模式 |
| **AI Security（AI 安全）** | 安全钩子、护栏 |
| **Optimization（优化）** | 性能、成本/token 缩减 |

若演讲有领域专属区域，可调整或新增类别。

## 输出格式

### concepts.md

```markdown
# Key Concepts — {provisional title}

**Date**: {date}
**Source**: {source path} × Summary × Timeline (if available)

---

## Concept table

| # | Concept | Category | Short description | Talk potential |
|---|---------|----------|------------------|----------------|
| 1 | **{Concept name}** | {Category} | {1-2 concrete sentences} | HIGH / MEDIUM / LOW |
...

---

## Category breakdown

| Category | Count | HIGH concepts | Examples |
|----------|-------|---------------|---------|
| {category} | {n} | {n} | {examples} |
...
| **TOTAL** | **{N}** | **{N HIGH}** | |

---

## Recommendations for positioning

{3-5 sentences on concept clusters that could form the talk's acts.
Which HIGH concepts reinforce each other? What narrative arc is emerging?}
```

### concepts-enriched.md（若代码库可用）

结构相同，但聚焦于代码库分析揭示的内容：
- 专用智能体（数量、规模、角色）
- 可调用技能（目录、覆盖领域）
- 系统钩子（事件、逻辑）
- 模块化配置（配置文件、模块、流水线）
- 项目专属代码模式

每个增强概念需包含：
- **精确来源**：文件及大致行号
- **可演示**：是/否（能否在幻灯片或现场展示？）

## 反模式

- 创建过于细粒度的概念（一个功能最多对应一个概念）
- 默认评分为 HIGH——需有选择性
- 遗漏 LOW 概念（它们在定位阶段作为"应避免的角度"很有价值）
- 重复非常相似的概念（应合并）
- 在代码库不可访问时分析代码库代码

## 验证清单

- [ ] 至少识别出15个概念（带代码库的 REX 需20个以上）
- [ ] 每个概念有1-2句具体描述
- [ ] 评分经过校准（不能全是 HIGH，也不能全是 LOW）
- [ ] 类别覆盖了摘要的主题
- [ ] 包含定位建议
- [ ] 文件保存到正确路径

## 提示

- 概念目录是第4阶段（定位）的输入来源——目录越丰富，角度选择越好
- LOW 概念很有价值：它们划定了"不应放入演讲"的边界
- 如果两个概念感觉非常相似，合并它们——更精简、更锐利的列表胜过冗长稀释的列表

## 相关

- [第1阶段：提取](../stage-1-extract/SKILL.md) — 前置条件
- [第2阶段：研究](../stage-2-research/SKILL.md) — 提供时间线（REX）
- [第4阶段：定位](../stage-4-position/SKILL.md) — 读取此目录
- [编排器](../orchestrator/SKILL.md)
