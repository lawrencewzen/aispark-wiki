> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Claude Code 工作流"
description: "使用 Claude Code 进行常见开发模式的分步指南"
tags: [workflow, guide, reference]
---

# Claude Code 工作流

使用 Claude Code 进行常见开发模式的分步指南。

---

## 搜索与发现

### [搜索工具精通](./search-tools-mastery.md) ⭐ 新增

**掌握结合 rg、grepai、Serena 和 ast-grep 的代码搜索艺术**

学习何时使用每种工具、如何将它们结合以实现最高效率，以及真实世界的工作流，包括：
- 探索陌生代码库
- 大规模重构
- 安全审计
- 框架迁移
- 性能优化

**核心主题**：
- 快速决策矩阵
- 完整功能对比
- 5 种组合工作流
- 性能基准测试
- 常见陷阱
- 工具选择速查表

---

## 开发工作流

### [计划驱动开发](./plan-driven.md)

在执行之前用计划模式构建复杂任务。

**适用场景**：多步骤功能、架构变更、对方案不确定时

### [使用 Claude 进行 TDD（测试驱动开发）](./tdd-with-claude.md)

测试驱动开发工作流：先写测试，再实现。

**适用场景**：关键功能、防止回退、API 设计

### [规格优先开发](./spec-first.md)

先写规格再写代码，获得更好的需求清晰度。

**适用场景**：团队协作、复杂功能、文档优先项目

### [迭代优化](./iterative-refinement.md)

通过多轮优化循环改善代码。

**适用场景**：质量提升、性能优化、代码清理

### [骨架项目](./skeleton-projects.md) ⭐ 新增

使用现有的经过实战检验的仓库作为新项目的脚手架。

**适用场景**：启动新项目、统一团队模式、从成熟基础进行快速原型开发

### [团队 AI 指令](./team-ai-instructions.md)

通过基于配置文件的模块化组装，跨多开发者多工具团队统一管理 CLAUDE.md。

**适用场景**：5 人以上团队、多种 AI 工具（Claude Code + Cursor/Windsurf）、混合操作系统

### [Changelog 片段](./changelog-fragments.md) ⭐ 新增

**通过三层系统强制执行每 PR 文档化：CLAUDE.md 规则 + UserPromptSubmit 钩子 + CI 门控**

消除 `CHANGELOG.md` 上的合并冲突，在实现时捕获上下文，确保 DB 迁移不会被静默部署。包含可复用的 `UserPromptSubmit` 钩子模式，用于强制执行任何必须遵守的工作流步骤。

**核心主题**：
- 用于自主片段创建的 CLAUDE.md 工作流规则
- 带三层优先级的 `UserPromptSubmit` 钩子（强制执行、发现、上下文相关）
- 条件建议模式：「如果有 PR 意图但没有提到片段」
- 带独立迁移检查 job 的 CI 强制执行

### [RPI：研究 → 计划 → 实现](./rpi.md) ⭐ 新增

**三阶段功能开发，各阶段之间有明确的验证门控**

三个锁定阶段构建功能：先研究可行性，再规划实现，最后编写代码。每个阶段产生一个具体产物（RESEARCH.md → PLAN.md → 代码）。每个门控需要明确的 GO 才能启动下一阶段。

**适用场景**：可行性不明确的功能、超过一天的工作、未知技术领域，或任何后期发现错误假设代价高昂的情况

### [GitHub Actions 工作流](./github-actions.md) ⭐ 新增

**5 种用于自动化 PR 审查、Issue 分类和质量门控的生产就绪模式**

通过官方 `claude-code-action` 将 Claude 直接连接到你的 GitHub 工作流。两种模式：交互式（`@claude` 提及）和完全自动化（推送/定时触发）。

**核心主题**：
- 通过 `/install-github-app` 快速配置（30 秒）
- 模式 1：通过 `@claude` 提及按需 PR 审查
- 模式 2：每次推送时自动审查
- 模式 3：Issue 分类和打标签
- 模式 4：对敏感路径的安全专项审查
- 模式 5：定时每周仓库健康检查
- 成本控制、并发、fork 安全

**适用场景**：任何希望获得 AI 驱动代码审查而无需管理基础设施的团队

---

### [认知模式切换](./gstack-workflow.md) ⭐ 新增

在整个发布周期中切换专业角色：战略产品门控、架构审查、偏执代码审查、自动化发布、原生浏览器 QA 和复盘。

**适用场景**：希望在产品方向、工程严谨度、审查和发布之间明确分离的发布周期——而非由一个通用助手处理所有阶段

---

## 设计与内容

### [设计转代码](./design-to-code.md)

将设计稿（Figma、线框图）转换为可运行的代码。

**适用场景**：前端开发、UI 实现、设计系统工作

### [OG 图片生成](./og-image-generation.md)

在构建时用 Satori 和 resvg 动态生成社交预览图。

**适用场景**：Astro 项目，在不维护静态 PNG 的情况下保持社交预览准确

### [PDF 生成](./pdf-generation.md)

使用 Quarto/Typst 配合 Claude Code 生成专业 PDF。

**适用场景**：报告、文档、白皮书、技术文档

### [演讲准备流水线](./talk-pipeline.md) ⭐ 新增

六阶段 Skills（技能模块）流水线：原始素材 → 结构化演讲 → 通过 Kimi 生成的 AI 幻灯片。

**适用场景**：会议演讲、技术 Meetup、内部技术分享——来源于文章、对话记录或笔记

### [TTS 配置](./tts-setup.md)

为 Claude Code 响应配置文字转语音（Agent Vibes 集成）。

**适用场景**：音频反馈、无障碍访问、免手动编码

---

## 代码探索

### [探索工作流](./exploration-workflow.md)

系统性地探索和理解陌生代码库。

**适用场景**：新项目、遗留代码、文档缺失

**相关**：高级多工具探索策略参见[搜索工具精通](./search-tools-mastery.md)。

---

## 多智能体与进阶

### [智能体团队](./agent-teams.md)

编排多个专业智能体并行处理复杂任务。

**适用场景**：受益于并行化、专业专长或独立验证的任务

### [智能体团队快速入门](./agent-teams-quick-start.md)

在 30 分钟内建立第一个智能体团队的快速指南。

**适用场景**：多智能体模式新手，想在完整配置前先实验

### [双实例规划](./dual-instance-planning.md)

在两个协调的 Claude Code 实例中，用 Opus 规划，用 Sonnet 执行。

**适用场景**：需要深度推理架构的复杂功能，成本效益执行

### [事件驱动智能体](./event-driven-agents.md)

通过钩子事件协调智能体，而非直接编排。

**适用场景**：响应式工作流、钩子触发的自动化、松耦合的智能体流水线

### [计划流水线](./plan-pipeline.md)

完整的端到端计划流水线：/plan-start、/plan-validate、/plan-execute 作为一个连贯工作流。

**适用场景**：在编写代码之前计划严谨度有价值的任何重大功能

### [任务管理](./task-management.md)

使用 TodoWrite、Tasks API 和跨会话上下文持久化进行多会话任务追踪。

**适用场景**：跨多个会话的长期任务、团队协调、复杂的任务积压

---

## 快速选择指南

| 你的情况 | 推荐工作流 |
|----------------|---------------------|
| **代码库陌生** | [探索工作流](./exploration-workflow.md) + [搜索工具精通](./search-tools-mastery.md) |
| **复杂功能** | [计划驱动](./plan-driven.md) 或 [规格优先](./spec-first.md) |
| **需要可靠性** | [使用 Claude 进行 TDD（测试驱动开发）](./tdd-with-claude.md) |
| **大规模重构** | [搜索工具精通](./search-tools-mastery.md) |
| **UI 实现** | [设计转代码](./design-to-code.md) |
| **代码质量** | [迭代优化](./iterative-refinement.md) |
| **从模板新建项目** | [骨架项目](./skeleton-projects.md) |
| **团队 AI 指令** | [团队 AI 指令](./team-ai-instructions.md) |
| **强制必须遵守的工作流步骤** | [Changelog 片段](./changelog-fragments.md) |
| **未知可行性，多天功能** | [RPI：研究 → 计划 → 实现](./rpi.md) |
| **文档** | [PDF 生成](./pdf-generation.md) |
| **社交预览** | [OG 图片生成](./og-image-generation.md) |
| **从原始素材准备会议演讲** | [演讲准备流水线](./talk-pipeline.md) |
| **音频反馈** | [TTS 配置](./tts-setup.md) |
| **多智能体任务** | [智能体团队](./agent-teams.md) |
| **第一个智能体团队** | [智能体团队快速入门](./agent-teams-quick-start.md) |
| **成本优化规划** | [双实例规划](./dual-instance-planning.md) |
| **钩子驱动的自动化** | [事件驱动智能体](./event-driven-agents.md) |
| **完整计划工作流** | [计划流水线](./plan-pipeline.md) |
| **多会话追踪** | [任务管理](./task-management.md) |
| **编码前的战略门控** | [认知模式切换](./gstack-workflow.md) |
| **非 MCP 浏览器自动化** | [认知模式切换](./gstack-workflow.md) |

---

## 贡献

有新的工作流想法？在主仓库中开 Issue 或 PR。

**工作流模板结构**：
1. 标题与目的
2. 适用场景
3. 前置条件
4. 分步指南
5. 真实案例
6. 常见陷阱
7. 相关工作流

---

**最后更新**：2026 年 3 月
