> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Claude Code 示例"
description: "带注释的模板，讲解模式为何有效，并附权衡分析与替代方案"
tags: [template, reference, workflows, architecture]
---

# Claude Code 示例

带注释的模板，帮助你理解模式**为何**有效，而不仅仅是如何配置。每个模板都包含注释，解释权衡取舍、替代方案以及何时应当偏离默认做法。

> **[📚 浏览自动生成的目录](./CATALOG.md)** — 按复杂度、时间、领域索引（181 个模板）  
> **[🔍 浏览交互式目录](./index.html)** — 带语法高亮的模板查看、复制与下载

## 新功能：自动生成目录

**[`CATALOG.md`](./CATALOG.md)** 现已从模板元数据自动生成，按以下维度组织：
- **复杂度**：初级、中级、高级
- **时间**：5 分钟至 4 小时以上
- **领域**：安全、测试、部署、性能、架构、自动化
- **关键词**：可搜索的标签，便于发现

每个模板均包含元数据：
```yaml
---
name: template-name
description: One-line description
complexity: beginner|intermediate|advanced
time: 5 min|15 min|30 min|1 hour|2 hours|4+ hours|varies
domain: security|testing|deployment|etc
prerequisites: []
status: stable|experimental|deprecated
keywords: [tag1, tag2]
---
```

**参见 [`docs/template-metadata-schema.md`](../docs/template-metadata-schema.md)** 了解完整规范。

## 目录结构

| 文件夹 | 描述 | 数量 |
|--------|------|------|
| [`agents/`](./agents/) | 针对特定任务的自定义 AI 角色 | 21 + 2 个集合 |
| [`commands/`](./commands/) | 斜杠命令（工作流自动化） | 52 |
| [`hooks/`](./hooks/) | 事件驱动的安全与自动化脚本 | 37 |
| [`skills/`](./skills/) | 可复用知识模块 — [9 个在 SkillHub](https://) | 68 |
| [`claude-md/`](./claude-md/) | CLAUDE.md 配置文件 | 7 |
| [`config/`](./config/) | 设置、MCP、git 模板 | 6 |
| [`memory/`](./memory/) | CLAUDE.md 记忆文件模板 | 1 |
| [`rules/`](./rules/) | 常见代码审查模式的行为规则 | 5 |
| [`scripts/`](./scripts/) | 诊断与实用脚本 | 17 |
| [`team-config/`](./team-config/) | 团队入职模板 | 3 |
| [`templates/`](./templates/) | 会话与工作流模板 | 1 |
| [`github-actions/`](./github-actions/) | CI/CD 工作流 | 6 |
| [`workflows/`](./workflows/) | 高级开发工作流 | 3 |
| [`plugins/`](./plugins/) | 社区插件（SE-CoVe、claude-mem） | 2 |
| [`integrations/`](./integrations/) | 外部工具集成（Agent Vibes TTS） | 3 |
| [`context-engineering/`](./context-engineering/) | 上下文工程模式与配置文件 | 10 |
| [`mcp-configs/`](./mcp-configs/) | MCP 服务器配置 | 1 |
| [`modes/`](./modes/) | 行为模式（SuperClaude） | 1 |
| [`semantic-anchors/`](./semantic-anchors/) | 精确词汇表，提升 LLM 输出质量 | 1 |
| [`multi-provider/`](https://github.com/claude-code-ultimate-guide/cc-copilot-bridge) | 多提供商桥接 → 独立仓库 | — |

## 快速开始

1. 复制所需模板
2. 按项目需求自定义
3. 放置到正确位置（见下方路径说明）

## 文件位置

| 类型 | 项目位置 | 全局位置 |
|------|----------|----------|
| 智能体 | `.claude/agents/` | `~/.claude/agents/` |
| 技能 | `.claude/skills/` | `~/.claude/skills/` |
| 命令 | `.claude/commands/` | `~/.claude/commands/` |
| 钩子 | `.claude/hooks/` | `~/.claude/hooks/` |
| 配置 | `.claude/` | `~/.claude/` |
| 记忆 | `./CLAUDE.md` 或 `.claude/CLAUDE.md` | `~/.claude/CLAUDE.md` |
| 模式 | — | `~/.claude/MODE_*.md` |

> **Windows**：将 `~/.claude/` 替换为 `%USERPROFILE%\.claude\`

## 模板索引

### 智能体（23 个）

| 文件 | 用途 | 模型 |
|------|------|------|
| [code-reviewer.md](./agents/code-reviewer.md) | 全面的代码审查 | Sonnet |
| [test-writer.md](./agents/test-writer.md) | TDD/BDD 测试生成 | Sonnet |
| [security-auditor.md](./agents/security-auditor.md) | 安全漏洞检测 | Sonnet |
| [refactoring-specialist.md](./agents/refactoring-specialist.md) | 代码整洁重构 | Sonnet |
| [output-evaluator.md](./agents/output-evaluator.md) | LLM 作为裁判的质量关卡 | Haiku |
| [devops-sre.md](./agents/devops-sre.md) | 基于 FIRE 框架的基础设施故障排查 | Sonnet |
| [planner.md](./agents/planner.md) | 战略规划 — 只读，在实现之前使用 | Opus |
| [implementer.md](./agents/implementer.md) | 机械执行 — 有界范围 | Haiku |
| [architecture-reviewer.md](./agents/architecture-reviewer.md) | 架构与设计审查 — 只读 | Opus |
| [adr-writer.md](./agents/adr-writer.md) | 架构决策记录生成器 — 只读 | Opus |
| [integration-reviewer.md](./agents/integration-reviewer.md) | 运行时集成验证器 — 只读 | Sonnet |
| [plan-challenger.md](./agents/plan-challenger.md) | 跨 5 个维度的对抗性计划审查 — 只读 | Sonnet |
| [planning-coordinator.md](./agents/planning-coordinator.md) | 动态研究团队的综合智能体 — 只读 | Sonnet |
| [security-patcher.md](./agents/security-patcher.md) | 应用审计发现的安全补丁 — 提交审查 | Sonnet |
| [analytics-with-eval/](./agents/analytics-with-eval/) | 集合：分析智能体 + 评估钩子 | — |
| [cyber-defense/](./agents/cyber-defense/) | 集合：异常检测器、日志摄取器、风险分类器、威胁报告器 | — |

### 技能（68 个）— [9 个在 SkillHub](https://)

| 文件 | 用途 |
|------|------|
| [git-ai-archaeology/](./skills/git-ai-archaeology/) | 分析 git 仓库中 AI 配置的演变 — 每个路径的首次提交、月度分布、重大 PR、成熟度阶段 |
| [token-audit/](./skills/token-audit/) | 测量固定上下文的令牌开销，按使用频率分类规则，审计钩子成本，生成优先级行动计划 |
| [design-patterns/](./skills/design-patterns/) | 检测并分析 GoF 设计模式，提供堆栈感知的建议 |
| [tdd-workflow.md](./skills/tdd-workflow.md) | 测试驱动开发流程 |
| [security-checklist.md](./skills/security-checklist.md) | OWASP Top 10 安全检查 |
| [pdf-generator.md](./skills/pdf-generator.md) | 专业 PDF 生成（Quarto/Typst） |
| [voice-refine/](./skills/voice-refine/) | 写作风格精炼，含前后对比示例 |
| [ast-grep-patterns.md](./skills/ast-grep-patterns.md) | 基于 AST 的代码搜索模式 |
| [rtk-optimizer/](./skills/rtk-optimizer/) | RTK 令牌优化分析 |
| [audit-agents-skills/](./skills/audit-agents-skills/) | 智能体、技能和命令的质量审计 |
| [skill-creator/](./skills/skill-creator/) | 创建具有正确结构和最佳实践的新技能 |
| [landing-page-generator/](./skills/landing-page-generator/) | 从任意仓库生成即部署的落地页 |
| [ccboard/](./skills/ccboard/) | Claude Code 监控的综合 TUI/Web 仪表板 |
| [guide-recap/](./skills/guide-recap/) | 将 CHANGELOG 条目转换为社交内容（LinkedIn、Twitter/X、Slack） |
| [release-notes-generator/](./skills/release-notes-generator/) | 从 git 提交生成 3 种格式的发布说明 |
| [pr-triage/](./skills/pr-triage/) | 4 阶段 PR 积压管理（审计、深度审查、验证评论、工作树设置） |
| [issue-triage/](./skills/issue-triage/) | 3 阶段 Issue 积压管理（审计、深度分析、验证操作） |
| [cyber-defense-team/](./skills/cyber-defense-team/) | 多智能体网络防御团队编排 |
| [talk-pipeline/](./skills/talk-pipeline/) | 6 阶段流水线：从原始素材到通过 Kimi 生成幻灯片 |
| [eval-rules/](./skills/eval-rules/) | 审计 `.claude/rules/` 文件 — 解析 glob 模式以匹配实际项目文件，交互式实用性审查，原地编辑 |

### 命令（52 个）

| 文件 | 触发器 | 用途 |
|------|--------|------|
| [commit.md](./commands/commit.md) | `/commit` | 约定式提交消息 |
| [pr.md](./commands/pr.md) | `/pr` | 创建带范围分析的结构化 PR |
| [review-pr.md](./commands/review-pr.md) | `/review-pr` | PR 审查工作流 |
| [release-notes.md](./commands/release-notes.md) | `/release-notes` | 生成 3 种格式的发布说明 |
| [sonarqube.md](./commands/sonarqube.md) | `/sonarqube` | 分析 PR 的 SonarCloud 质量问题 |
| [generate-tests.md](./commands/generate-tests.md) | `/generate-tests` | 测试生成 |
| [git-worktree.md](./commands/git-worktree.md) | `/git-worktree` | 隔离的 git 工作树设置 |
| [git-worktree-status.md](./commands/git-worktree-status.md) | `/git-worktree-status` | 检查工作树后台验证任务 |
| [git-worktree-remove.md](./commands/git-worktree-remove.md) | `/git-worktree-remove` | 带合并检查的安全工作树删除 |
| [git-worktree-clean.md](./commands/git-worktree-clean.md) | `/git-worktree-clean` | 批量清理过期工作树 |
| [diagnose.md](./commands/diagnose.md) | `/diagnose` | 交互式故障排查助手（法语/英语） |
| [validate-changes.md](./commands/validate-changes.md) | `/validate-changes` | LLM 作为裁判的提交前验证 |
| [catchup.md](./commands/catchup.md) | `/catchup` | 执行 /clear 后恢复上下文 |
| [security.md](./commands/security.md) | `/security` | 快速 OWASP 安全审计 |
| [security-check.md](./commands/security-check.md) | `/security-check` | 配置扫描（约 30 秒） |
| [security-audit.md](./commands/security-audit.md) | `/security-audit` | 完整 6 阶段审计，评分满分 100 |
| [update-threat-db.md](./commands/update-threat-db.md) | `/update-threat-db` | 研究并更新威胁情报 |
| [audit-agents-skills.md](./commands/audit-agents-skills.md) | `/audit-agents-skills` | `.claude/` 配置质量审计 |
| [sandbox-status.md](./commands/sandbox-status.md) | `/sandbox-status` | 沙盒隔离状态检查 |
| [refactor.md](./commands/refactor.md) | `/refactor` | 基于 SOLID 原则的代码改进 |
| [explain.md](./commands/explain.md) | `/explain` | 代码解释（3 个深度级别） |
| [optimize.md](./commands/optimize.md) | `/optimize` | 性能分析与路线图 |
| [ship.md](./commands/ship.md) | `/ship` | 部署前检查清单 |
| [learn/quiz.md](./commands/learn/quiz.md) | `/learn:quiz` | 学习概念的自测 |
| [learn/teach.md](./commands/learn/teach.md) | `/learn:teach` | 逐步讲解概念 |
| [learn/alternatives.md](./commands/learn/alternatives.md) | `/learn:alternatives` | 比较不同方法 |
| [audit-codebase.md](./commands/audit-codebase.md) | `/audit-codebase` | 代码库健康审计，对 7 个类别评分 |
| [plan-start.md](./commands/plan-start.md) | `/plan-start` | 5 阶段规划：PRD 分析、设计审查、技术决策、研究团队、指标 |
| [plan-execute.md](./commands/plan-execute.md) | `/plan-execute` | 执行已验证的计划：工作树隔离、TDD 脚手架、并行智能体、PR 创建 |
| [plan-validate.md](./commands/plan-validate.md) | `/plan-validate` | 2 层计划验证：结构检查 + 专家智能体，自动修复问题 |
| [review-plan.md](./commands/review-plan.md) | `/review-plan` | 编写代码前跨 4 个维度的结构化计划审查 |
| [check-cache-bugs.md](./commands/check-cache-bugs.md) | `/check-cache-bugs` | 审计 CC#40524 缓存 bug，这类 bug 可能静默地将 API 成本放大 10-20 倍 |

### 钩子（37 个）

安全优先：12 个安全钩子、8 个生产力钩子、5 个自动化钩子、5 个监控钩子。

**安全钩子**（13 个 bash）：

| 文件 | 事件 | 用途 |
|------|------|------|
| [dangerous-actions-blocker.sh](./hooks/bash/dangerous-actions-blocker.sh) | PreToolUse | 阻止 `rm -rf`、强制推送、生产操作 |
| [prompt-injection-detector.sh](./hooks/bash/prompt-injection-detector.sh) | PreToolUse | 检测提示词中的注入模式 |
| [unicode-injection-scanner.sh](./hooks/bash/unicode-injection-scanner.sh) | PreToolUse | 检测零宽字符、RTL 覆盖、ANSI 转义 |
| [repo-integrity-scanner.sh](./hooks/bash/repo-integrity-scanner.sh) | PreToolUse | 扫描 README/package.json 中的隐藏注入 |
| [security-check.sh](./hooks/bash/security-check.sh) | PreToolUse | 阻止命令中的密钥泄露 |
| [sandbox-validation.sh](./hooks/bash/sandbox-validation.sh) | PreToolUse | 验证沙盒隔离 |
| [file-guard.sh](./hooks/bash/file-guard.sh) | PreToolUse | 保护敏感文件不被修改 |
| [permission-request.sh](./hooks/bash/permission-request.sh) | PreToolUse | 高风险操作的显式权限流程 |
| [mcp-config-integrity.sh](./hooks/bash/mcp-config-integrity.sh) | SessionStart | 验证 MCP 配置哈希（CVE 防护） |
| [claudemd-scanner.sh](./hooks/bash/claudemd-scanner.sh) | SessionStart | 检测 CLAUDE.md 注入攻击 |
| [output-secrets-scanner.sh](./hooks/bash/output-secrets-scanner.sh) | PostToolUse | 防止 API 密钥/令牌出现在 Claude 响应中 |
| [pre-commit-secrets.sh](./hooks/bash/pre-commit-secrets.sh) | Git hook | 阻止密钥进入提交 |
| [security-gate.sh](./hooks/bash/security-gate.sh) | PreToolUse | 在写入源文件前检测易受攻击的代码模式 |

**生产力钩子**（10 个）：

| 文件 | 事件 | 用途 |
|------|------|------|
| [auto-format.sh](./hooks/bash/auto-format.sh) | PostToolUse | 编辑后自动格式化（Prettier、Black、go fmt） |
| [auto-checkpoint.sh](./hooks/bash/auto-checkpoint.sh) | PostToolUse | 定时自动保存检查点 |
| [typecheck-on-save.sh](./hooks/bash/typecheck-on-save.sh) | PostToolUse | 保存时运行 TypeScript 检查 |
| [test-on-change.sh](./hooks/bash/test-on-change.sh) | PostToolUse | 文件变更时运行测试 |
| [rtk-auto-wrapper.sh](./hooks/bash/rtk-auto-wrapper.sh) | PreToolUse | 自动用 RTK 包装命令以节省令牌 |
| [rtk-baseline.sh](./hooks/bash/rtk-baseline.sh) | SessionStart | 保存 RTK 基线，用于追踪会话节省量 |
| [setup-init.sh](./hooks/bash/setup-init.sh) | SessionStart | 初始化会话环境 |
| [subagent-stop.sh](./hooks/bash/subagent-stop.sh) | Stop | 清理子智能体资源 |
| [auto-rename-session.sh](./hooks/bash/auto-rename-session.sh) | SessionEnd | AI 驱动的会话标题生成（Haiku） |
| [velocity-governor.sh](./hooks/bash/velocity-governor.sh) | PreToolUse | 限制工具调用速率以避免 API 限流 |

**监控钩子**（6 个）：

| 文件 | 事件 | 用途 |
|------|------|------|
| [output-validator.sh](./hooks/bash/output-validator.sh) | PostToolUse | 启发式输出验证 |
| [session-logger.sh](./hooks/bash/session-logger.sh) | PostToolUse | 记录操作日志用于监控 |
| [session-summary.sh](./hooks/bash/session-summary.sh) | SessionEnd | 显示会话统计（时长、工具使用、成本、RTK 节省） |
| [session-summary-config.sh](./hooks/bash/session-summary-config.sh) | CLI 工具 | 配置会话摘要的区块和显示方式 |
| [learning-capture.sh](./hooks/bash/learning-capture.sh) | Stop | 提示记录每日学习内容 |
| [privacy-warning.sh](./hooks/bash/privacy-warning.sh) | PostToolUse | 警告潜在的隐私泄露 |

**通知与 TTS**（3 个）：

| 文件 | 事件 | 用途 |
|------|------|------|
| [notification.sh](./hooks/bash/notification.sh) | Notification | 上下文感知的 macOS 声音提醒 |
| [tts-selective.sh](./hooks/bash/tts-selective.sh) | PostToolUse | 对选定输出进行文字转语音 |
| [pre-commit-evaluator.sh](./hooks/bash/pre-commit-evaluator.sh) | Git hook | LLM 作为裁判的提交前评估 |

**PowerShell**（2 个）：

| 文件 | 事件 | 用途 |
|------|------|------|
| [security-check.ps1](./hooks/powershell/security-check.ps1) | PreToolUse | 阻止命令中的密钥泄露 |
| [auto-format.ps1](./hooks/powershell/auto-format.ps1) | PostToolUse | 编辑后自动格式化 |

> **参见 [hooks/README.md](./hooks/README.md) 了解完整文档、配置示例和安全加固模式**

### 配置（6 个）

| 文件 | 用途 |
|------|------|
| [settings.json](./config/settings.json) | 钩子配置 |
| [mcp.json](./config/mcp.json) | MCP 服务器设置 |
| [.gitignore-claude](./config/.gitignore-claude) | Git 忽略模式 |
| [CONTRIBUTING-ai-disclosure.md](./config/CONTRIBUTING-ai-disclosure.md) | CONTRIBUTING.md 的 AI 披露模板 |
| [PULL_REQUEST_TEMPLATE-ai.md](./config/PULL_REQUEST_TEMPLATE-ai.md) | 带 AI 署名的 PR 模板 |
| [sandbox-native.json](./config/sandbox-native.json) | Claude Code 原生沙盒配置 |
| [settings-personalization.json](./config/settings-personalization.json) | UI 个性化：加载动画动词、自定义提示轮播 |
| [settings.local.json.example](./config/settings.local.json.example) | 本地覆盖示例（已 gitignore） |

### 记忆（1 个）

| 文件 | 用途 |
|------|------|
| [CLAUDE.md.project-template](./memory/CLAUDE.md.project-template) | 团队项目记忆 |
| [CLAUDE.md.personal-template](./memory/CLAUDE.md.personal-template) | 个人全局记忆 |

### CLAUDE.md 配置（7 个）

| 文件 | 用途 |
|------|------|
| [learning-mode.md](./claude-md/learning-mode.md) | 以学习为中心的开发配置 |
| [devops-sre.md](./claude-md/devops-sre.md) | DevOps/SRE 项目配置 |
| [product-designer.md](./claude-md/product-designer.md) | 产品设计师工作流配置 |
| [tts-enabled.md](./claude-md/tts-enabled.md) | 启用文字转语音的配置 |
| [rtk-optimized.md](./claude-md/rtk-optimized.md) | RTK 令牌优化配置 |
| [session-naming.md](./claude-md/session-naming.md) | 自动为并行工作的会话重命名为描述性标题 |
| [design-reference-file.md](./claude-md/design-reference-file.md) | 品牌手册和 UI 套件上下文，用于一致的 UI 生成 |

> **参见 [guide/learning-with-ai.md](../guide/roles/learning-with-ai.md) 了解学习模式文档**  
> **参见 [guide/devops-sre.md](../guide/ops/devops-sre.md) 了解 DevOps/SRE 指南**

### 脚本（17 个）

| 文件 | 用途 | 输出 |
|------|------|------|
| [audit-scan.sh](./scripts/audit-scan.sh) | 快速设置审计扫描器 | JSON / 人类可读 |
| [check-claude.sh](./scripts/check-claude.sh) | 健康检查诊断（macOS/Linux） | 人类可读 |
| [check-claude.ps1](./scripts/check-claude.ps1) | 健康检查诊断（Windows） | 人类可读 |
| [clean-reinstall-claude.sh](./scripts/clean-reinstall-claude.sh) | 干净重装流程（macOS/Linux） | 人类可读 |
| [clean-reinstall-claude.ps1](./scripts/clean-reinstall-claude.ps1) | 干净重装流程（Windows） | 人类可读 |
| [session-stats.sh](./scripts/session-stats.sh) | 分析会话日志与成本 | JSON / 人类可读 |
| [session-search.sh](./scripts/session-search.sh) | 快速会话搜索与恢复 | 人类可读 |
| [cc-sessions.py](./scripts/cc-sessions.py) | 带增量索引的高级会话搜索 | 人类可读 |
| [fresh-context-loop.sh](./scripts/fresh-context-loop.sh) | 在上下文限制时自动重启会话 | 人类可读 |
| [bridge.py](./scripts/bridge.py) | 跨会话的计划桥接 | JSON |
| [bridge-plan-schema.json](./scripts/bridge-plan-schema.json) | bridge plan v1 格式的 JSON Schema | — |
| [migrate-arguments-syntax.sh](./scripts/migrate-arguments-syntax.sh) | 迁移 v1 → v2 参数语法（bash） | 人类可读 |
| [migrate-arguments-syntax.ps1](./scripts/migrate-arguments-syntax.ps1) | 迁移 v1 → v2 参数语法（PowerShell） | 人类可读 |
| [rtk-benchmark.sh](./scripts/rtk-benchmark.sh) | 基准测试 RTK 令牌节省量 | 人类可读 |
| [sync-claude-config.sh](./scripts/sync-claude-config.sh) | 跨机器同步 Claude 配置 | 人类可读 |
| [sonnetplan.sh](./scripts/sonnetplan.sh) | 别名：使用 Sonnet 而非 Opus 运行 Claude（成本优化） | 人类可读 |

> **参见 [scripts/README.md](./scripts/README.md) 了解详细用法**

### 规则（5 个）

| 文件 | 用途 |
|------|------|
| [architecture-review.md](./rules/architecture-review.md) | 架构审查会话的规则 |
| [code-quality-review.md](./rules/code-quality-review.md) | 代码质量审查会话的规则 |
| [first-principles.md](./rules/first-principles.md) | 第一性原理推理规则 |
| [performance-review.md](./rules/performance-review.md) | 性能审查会话的规则 |
| [test-review.md](./rules/test-review.md) | 测试审查会话的规则 |

### 团队配置（3 个）

| 文件 | 用途 |
|------|------|
| [claude-skeleton.md](./team-config/claude-skeleton.md) | 新团队成员的最小化 CLAUDE.md 骨架 |
| [profile-template.yaml](./team-config/profile-template.yaml) | 多工具团队的配置文件组装模板 |
| [sync-script.ts](./team-config/sync-script.ts) | 跨团队机器同步 Claude 配置 |

### 模板（1 个）

| 文件 | 用途 |
|------|------|
| [session-handoff-lorenz.md](./templates/session-handoff-lorenz.md) | 保持上下文连续性的会话交接模板 |

### GitHub Actions（6 个）

| 文件 | 触发器 | 用途 |
|------|--------|------|
| [claude-code-review.yml](./github-actions/claude-code-review.yml) ⭐ | PR 打开/同步 + `/claude-review` 评论 | 基于提示词的审查（外置提示词 + 防幻觉协议） |
| [claude-pr-auto-review.yml](./github-actions/claude-pr-auto-review.yml) | PR 打开/更新 | 带行内评论的自动代码审查 |
| [claude-security-review.yml](./github-actions/claude-security-review.yml) | PR 打开/更新 | 安全扫描（OWASP） |
| [claude-issue-triage.yml](./github-actions/claude-issue-triage.yml) | Issue 创建 | 自动分类，含标签和严重级别 |

> **参见 [github-actions/README.md](./github-actions/README.md) 了解设置说明和自定义方法**

### 工作流（3 个）

| 文件 | 用途 |
|------|------|
| [database-branch-setup.md](./workflows/database-branch-setup.md) | 带数据库分支的隔离功能开发（Neon/PlanetScale） |
| [memory-stack-integration.md](./workflows/memory-stack-integration.md) | 使用记忆工具的多日工作流（claude-mem + Serena + grepai） |
| [remotion-quickstart.md](./workflows/remotion-quickstart.md) | 使用 Remotion 的视频生成工作流 |

### 插件（2 个）

| 文件 | 用途 |
|------|------|
| [se-cove.md](./plugins/se-cove.md) | 独立代码审查的验证链（Meta AI，ACL 2024） |
| [claude-mem.md](./plugins/claude-mem.md) | 持久记忆管理插件 |

### 集成（3 个）

| 工具 | 用途 |
|------|------|
| [Agent Vibes TTS](./integrations/agent-vibes/) | Claude Code 响应的文字转语音朗读 |

> **参见 [agent-vibes/README.md](./integrations/agent-vibes/README.md) 了解安装说明和语音目录**

### MCP 配置（1 个）

| 文件 | 用途 |
|------|------|
| [figma.json](./mcp-configs/figma.json) | Figma MCP 服务器配置 |

### 模式（1 个）

| 文件 | 用途 | 激活方式 |
|------|------|----------|
| [MODE_Learning.md](./modes/MODE_Learning.md) | 即时讲解 | `--learn` 标志 |

> **参见 [modes/README.md](./modes/README.md) 了解安装说明和 SuperClaude 框架参考**

### 语义锚点（1 个）

| 文件 | 用途 |
|------|------|
| [anchor-catalog.md](./semantic-anchors/anchor-catalog.md) | 用于提示词的精确技术术语综合目录 |

> **参见指南 [第 2.7 节](../guide/ultimate-guide.md#27-semantic-anchors)，了解如何使用语义锚点**

### 多提供商桥接

| 工具 | 用途 |
|------|------|
| [cc-copilot-bridge](https://github.com/claude-code-ultimate-guide/cc-copilot-bridge) | 将 GitHub Copilot 桥接到 Claude Code CLI，实现按月付费访问 |

> 已迁移至独立仓库：[github.com/claude-code-ultimate-guide/cc-copilot-bridge](https://github.com/claude-code-ultimate-guide/cc-copilot-bridge)

---

*参见[主指南](../guide/ultimate-guide.md)获取详细说明，或参见[架构指南](../guide/core/architecture.md)了解 Claude Code 内部工作原理。*
