> 📚 **AI Spark Wiki** · Claude Code 知识库

# 模板目录

自动生成的模板索引，支持按复杂度、时间和领域筛选。

**最后更新**：[自动生成]

---

**模板总数**：181

- **智能体**：23
- **命令**：52
- **技能**：64
- **钩子**：37
- **工作流**：3
- **脚本**：2

## 按复杂度筛选

- **初级**：0 个模板
- **中级**：181 个模板
- **高级**：0 个模板

## 按时间筛选

- **30 分钟**：181 个模板

---

## 按分类浏览

### 智能体（23）

- **[adr-writer](agents/adr-writer.md)** *中级* • 30 分钟
  架构决策记录生成智能体 — 只读。检测代码变更中的架构决策，对其重要性进行分级，并按照 Michael Nygard 的模式化 ADR 格式（上下文-决策-结果）生成架构决策记录。不修改代码。适合在重大变更后或需要记录某个决策时使用。

- **[analytics-agent](agents/analytics-with-eval/analytics-agent.md)** *中级* • 30 分钟
  内置评估和安全检查的 SQL 查询生成智能体

- **[anomaly-detector](agents/cyber-defense/anomaly-detector.md)** *中级* • 30 分钟
  从结构化安全事件中检测统计异常和攻击模式。网络防御流水线的第二阶段 — 读取 cyber-defense-events.json 并生成异常报告。

- **[architecture-reviewer](agents/architecture-reviewer.md)** *中级* • 30 分钟
  架构与设计审查智能体 — 只读。评估结构性决策，识别设计异味，并在实现前标记风险。不修改代码。适合在合并架构变更前或规划智能体生成计划后使用。

- **[code-reviewer](agents/code-reviewer.md)** *中级* • 30 分钟
  用于全面代码审查，涵盖质量、安全和性能检查

- **[devops-sre](agents/devops-sre.md)** *中级* • 30 分钟
  使用 FIRE 框架（首次响应、调查、修复、评估）进行基础设施故障排查

- **[implementer](agents/implementer.md)** *中级* • 30 分钟
  用于边界清晰、定义明确任务的机械执行智能体。任务提示中必须明确说明范围和方案。适合在规划智能体生成计划后使用。对于复杂逻辑或设计决策，请改用 Sonnet。

- **[integration-reviewer](agents/integration-reviewer.md)** *中级* • 30 min
  运行时集成校验智能体 — 只读。校验服务连接参数、异步/同步一致性、环境变量完整性、库 API 正确性以及 OTEL 流水线完整性。当 /plan-validate 涉及新服务、库或可观测性配置时自动触发。

- **[log-ingestor](agents/cyber-defense/log-ingestor.md)** *中级* • 30 分钟
  将原始日志解析为结构化安全事件。网络防御流水线的第一阶段 — 读取日志文件并提取类型化事件（错误、警告、认证失败、异常）。

- **[loop-monitor](agents/loop-monitor.md)** *中级* • 30 分钟
  自主循环监控智能体 — 检测长时间无人值守 Claude 会话中的停滞、token 失控和无限循环。在运行自主流水线时配合看门狗进程使用。

- **[output-evaluator](agents/output-evaluator.md)** *中级* • 30 分钟
  在提交/执行前评估 Claude Code 输出质量（LLM 作为评判者模式）

- **[plan-challenger](agents/plan-challenger.md)** *中级* • 30 分钟
  对抗性计划审查智能体 — 只读。从 5 个维度系统性地攻击实现计划，然后应用反驳推理消除误报。不修改代码。在提交任何重要实现计划之前使用。

- **[planner](agents/planner.md)** *中级* • 30 分钟
  战略规划智能体 — 在实现前进行只读探索。用于分解任务、分析代码库并生成详细计划。不修改文件。

- **[planning-coordinator](agents/planning-coordinator.md)** *中级* • 30 分钟
  动态研究团队的综合智能体 — 只读。接收所有专项研究智能体的报告，生成连贯、无冗余的实现计划。在 /plan-start 第 4 阶段选择 2 个及以上智能体时自动启动。

- **[README](agents/cyber-defense/README.md)** *中级* • 30 分钟
  一个在日志文件中检测安全威胁的 4 智能体流水线，基于 Claude Code Agent 原生构建

- **[README](agents/analytics-with-eval/README.md)** *中级* • 30 分钟
  生产就绪的分析智能体，具备自动化指标收集和安全校验功能

- **[refactoring-specialist](agents/refactoring-specialist.md)** *中级* • 30 分钟
  用于遵循 SOLID 原则和最佳实践进行整洁代码重构

- **[report-template](agents/analytics-with-eval/eval/report-template.md)** *中级* • 30 分钟
  用于评分分析智能体性能和准确性的月度评估模板

- **[risk-classifier](agents/cyber-defense/risk-classifier.md)** *中级* • 30 分钟
  从检测到的异常中分类整体风险等级。网络防御流水线的第三阶段 — 读取 cyber-defense-anomalies.json 并给出带理由的 CRITICAL/HIGH/MEDIUM/LOW 评级。

- **[security-auditor](agents/security-auditor.md)** *中级* • 30 分钟
  用于安全漏洞检测和 OWASP 合规检查

- **[security-patcher](agents/security-patcher.md)** *中级* • 30 分钟
  根据 security-auditor 的发现应用安全补丁。需要审计报告作为输入。始终向人工审核提交补丁建议 — 未经批准不自动应用。

- **[test-writer](agents/test-writer.md)** *中级* • 30 分钟
  用于遵循 TDD/BDD 原则生成全面测试

- **[threat-reporter](agents/cyber-defense/threat-reporter.md)** *中级* • 30 分钟
  生成人类可读的安全事件报告。网络防御流水线的最终阶段 — 读取全部三个 JSON 文件，为安全团队生成 Markdown 报告。


### 命令（52）

- **[audit-agents-skills](commands/audit-agents-skills.md)** *中级* • 30 分钟
  审计 Claude Code 项目中智能体、技能和命令的质量

- **[audit-codebase](commands/audit-codebase.md)** *中级* • 30 分钟
  代码库健康审计，对 7 个类别评分并生成改进计划

- **[autoresearch](commands/autoresearch.md)** *中级* • 30 分钟
  自主改进循环 — 扫描代码库指标，构建实验文件，运行智能体驱动的迭代直至指标改善

- **[canary](commands/canary.md)** *中级* • 30 分钟
  部署后监控 — 在部署后监测生产环境并对回归发出告警

- **[catchup](commands/catchup.md)** *中级* • 30 分钟
  执行 /clear 后通过汇总近期工作和项目状态来恢复上下文

- **[check-cache-bugs](commands/check-cache-bugs.md)** *中级* • 30 分钟
  审计 Claude Code 配置中的缓存 bug（CC#40524）— 哨兵标记、--resume/--continue、归因头部及 ArkNill B3/B4/B5

- **[ci:all](commands/ci/all.md)** *中级* • 30 分钟
  完整 CI/CD 流水线：运行本地测试、类型检查、推送分支并返回流水线 URL。开 PR 前只需运行这一个命令。

- **[ci:pipeline](commands/ci/pipeline.md)** *中级* • 30 分钟
  推送当前分支并返回流水线跟踪 URL（GitLab 或 GitHub Actions）

- **[ci:status](commands/ci/status.md)** *中级* • 30 分钟
  显示当前分支的流水线状态 — GitLab CI 或 GitHub Actions

- **[ci:tests](commands/ci/tests.md)** *中级* • 30 分钟
  运行当前仓库的测试套件 — 自动检测 Python（pytest/uv）、Node（vitest/pnpm）或 Rust（cargo test）

- **[commit](commands/commit.md)** *中级* • 30 分钟
  为已暂存的变更生成符合规范的提交信息

- **[create-handoff](commands/handoff/create-handoff.md)** *中级* • 30 分钟
  从当前会话生成结构化交接文档。捕获范围、含行号的相关文件、关键发现、已完成工作、当前状态、后续步骤和代码片段。在结束会话或将工作移交给其他智能体前使用。

- **[diagnose](commands/diagnose.md)** *中级* • 30 分钟
  Claude Code 问题的交互式故障排查助手

- **[explain](commands/explain.md)** *中级* • 30 分钟
  以可调节的深度级别解释代码、概念或系统行为

- **[generate-tests](commands/generate-tests.md)** *中级* • 30 分钟
  为指定代码生成全面测试

- **[git-worktree](commands/git-worktree.md)** *中级* • 30 分钟
  创建隔离的 Git 工作树，无需切换分支即可进行功能开发

- **[git-worktree-clean](commands/git-worktree-clean.md)** *中级* • 30 分钟
  清理过期的 Git 工作树，包含已合并分支检测和磁盘使用报告

- **[git-worktree-remove](commands/git-worktree-remove.md)** *中级* • 30 分钟
  安全移除 Git 工作树，包含分支清理和安全检查

- **[git-worktree-status](commands/git-worktree-status.md)** *中级* • 30 分钟
  检查在 Git 工作树中运行的后台校验任务状态

- **[investigate](commands/investigate.md)** *中级* • 30 分钟
  系统性根因调试 — 在编写任何修复方案前先找到原因

- **[land-and-deploy](commands/land-and-deploy.md)** *中级* • 30 分钟
  合并 PR、等待 CI、验证部署、运行金丝雀 — 完整的落地流水线

- **[learn-alternatives](commands/learn/alternatives.md)** *中级* • 30 分钟
  比较解决同一问题的不同方案

- **[learn-quiz](commands/learn/quiz.md)** *中级* • 30 分钟
  测试对最近编写或接受代码的理解程度

- **[learn-teach](commands/learn/teach.md)** *中级* • 30 分钟
  以递进深度逐步讲解某个概念

- **[methodology-advisor](commands/methodology-advisor.md)** *中级* • 30 分钟
  分析您的代码库并提出 3 个针对性问题，以推荐合适的 AI 辅助开发方法论组合

- **[optimize](commands/optimize.md)** *中级* • 30 分钟
  分析代码、查询或系统并提出性能改进建议

- **[plan-ceo-review](commands/plan-ceo-review.md)** *中级* • 30 分钟
  战略产品关卡 — 在编写任何代码之前，挑战需求简报，发掘隐藏在请求中的 10 星级产品

- **[plan-eng-review](commands/plan-eng-review.md)** *中级* • 30 分钟
  工程架构关卡 — 在编写实现代码之前锁定架构、图表、边界情况和测试矩阵

- **[plan-execute](commands/plan-execute.md)** *中级* • 30 分钟
  执行已验证的计划：工作树隔离、TDD 脚手架、基于层级的并行智能体、含冒烟测试的质量关卡、PR 创建与合并。全程处理直至 PR 合并。

- **[plan-start](commands/plan-start.md)** *中级* • 30 分钟
  5 阶段规划命令：PRD 分析、设计评审、技术决策、动态研究团队、指标。在编写任何代码之前生成完整实现计划和架构决策记录。

- **[plan-validate](commands/plan-validate.md)** *中级* • 30 分钟
  2 层计划验证：即时结构检查 + 基于触发器的专项智能体。使用架构决策记录和第一性原理自动修复问题。所有问题必须在执行前解决。

- **[pr](commands/pr.md)** *中级* • 30 分钟
  分析变更、检测范围问题并创建结构良好的 PR

- **[qa](commands/qa.md)** *中级* • 30 分钟
  对 Web 应用进行系统性 QA 测试 — 感知差异、分层处理，并带修复验证循环

- **[README](commands/ci/README.md)** *中级* • 30 分钟
  CI/CD 工作流的斜杠命令。自动检测技术栈（Python/Node/Rust）并同时支持 GitLab CI 和 GitHub Actions

- **[recipe-template](commands/recipe-template.md)** *中级* • 30 分钟
  用于实现结构化配方的命令模板：验证前置条件，然后执行编号步骤。复制此模板并替换占位内容。"上下文验证检查点"部分是核心模式 — 它强制 Claude 在开始前验证前置条件。

- **[refactor](commands/refactor.md)** *中级* • 30 分钟
  分析代码中的 SOLID 违规并提出针对性改进建议

- **[release-notes](commands/release-notes.md)** *中级* • 30 分钟
  从 Git 提交生成多种格式的发版说明

- **[resume-handoff](commands/handoff/resume-handoff.md)** *中级* • 30 分钟
  加载交接文档并从上一个会话停止的地方继续工作。解析范围、文件引用、已完成工作和后续步骤，确认理解后再继续。

- **[review-plan](commands/review-plan.md)** *中级* • 30 分钟
  在编写任何代码之前从 4 个维度进行结构化计划审查（灵感来自 Garry Tan 的工作流）

- **[review-pr](commands/review-pr.md)** *中级* • 30 分钟
  对 Pull Request 进行全面代码审查

- **[routines-discover](commands/routines-discover.md)** *中级* • 30 分钟
  分析当前项目，发现三种触发类型（定时、API、GitHub 事件）中的高价值 Routines 使用场景。用法：/routines-discover

- **[sandbox-status](commands/sandbox-status.md)** *中级* • 30 分钟
  显示原生沙箱状态、配置和近期违规记录

- **[scaffold](commands/scaffold.md)** *中级* • 30 分钟
  交互式辅导，通过 4-5 个问题判断您需要智能体、命令、技能、钩子还是规则，然后生成开箱即用的模板。用法：/scaffold（无参数 — 启动辅导会话）

- **[security](commands/security.md)** *中级* • 30 分钟
  聚焦 OWASP Top 10 漏洞的快速安全评估

- **[security-audit](commands/security-audit.md)** *中级* • 30 分钟
  包含评分安全态势评估的全面安全审计

- **[security-check](commands/security-check.md)** *中级* • 30 分钟
  针对已知威胁数据库的快速配置安全检查

- **[session-save](commands/session-save.md)** *中级* • 30 分钟
  将当前会话状态 — 决策、修改的文件、当前状态和后续步骤 — 保存到交接文件以便后续恢复。

- **[ship](commands/ship.md)** *中级* • 30 分钟
  全面的部署前验证，确保发布就绪

- **[sonarqube](commands/sonarqube.md)** *中级* • 30 分钟
  分析特定 PR 的 SonarCloud 质量问题

- **[update-handoff](commands/handoff/update-handoff.md)** *中级* • 30 分钟
  用当前会话进度更新现有交接文档。按章节应用特定合并规则：已完成工作仅追加（不删除历史），状态和后续步骤替换，文件和发现合并。若找不到源文件则回退到创建新交接文档。

- **[update-threat-db](commands/update-threat-db.md)** *中级* • 30 分钟
  研究并更新 AI 智能体安全威胁情报数据库

- **[validate-changes](commands/validate-changes.md)** *中级* • 30 分钟
  在提交前使用 LLM 作为评判者模式评估已暂存的变更


### 技能（64）

- **[ast-grep-patterns](skills/ast-grep-patterns.md)** *中级* • 30 分钟
  教导 Claude 何时以及如何使用 ast-grep 进行结构化代码搜索的技能

- **[audit-agents-skills](skills/audit-agents-skills/SKILL.md)** *中级* • 30 分钟
  审计 Claude Code 智能体、技能和命令的质量与生产就绪程度。适合评估技能质量、检查生产就绪评分或将智能体与最佳实践模板对比时使用。

- **[before-after](skills/voice-refine/examples/before-after.md)** *中级* • 30 分钟
  将冗长语音输入转换为结构化提示的真实案例

- **[behavioral](skills/design-patterns/reference/behavioral.md)** *中级* • 30 分钟
  观察者、策略、命令、责任链等行为模式参考

- **[ccboard](skills/ccboard/SKILL.md)** *中级* • 30 分钟
  启动并导航 ccboard TUI/Web 仪表盘以监控 Claude Code。适合监控 token 用量、跟踪费用、浏览会话或检查跨项目 MCP 服务器状态时使用。

- **[ccboard-install](skills/ccboard/commands/install.md)** *中级* • 30 分钟
  安装或更新 ccboard

- **[ccboard-web](skills/ccboard/commands/web.md)** *中级* • 30 分钟
  启动 ccboard Web 界面

- **[changelog-parsing-rules](skills/guide-recap/references/changelog-parsing-rules.md)** *中级* • 30 分钟
  如何从 `CHANGELOG.md` 中提取和分类条目以生成社交内容。

- **[changelog-template](skills/release-notes-generator/assets/changelog-template.md)** *中级* • 30 分钟
  用于生成 CHANGELOG.md 条目的模板。

- **[commit-categories](skills/release-notes-generator/references/commit-categories.md)** *中级* • 30 分钟
  本文档定义如何根据规范化提交格式对提交进行分类。

- **[content-transformation](skills/guide-recap/references/content-transformation.md)** *中级* • 30 分钟
  将技术性 CHANGELOG 语言映射为面向用户的社交价值。将 tone-guidelines.md 的规则应用于所有内容

- **[costs](skills/ccboard/commands/costs.md)** *中级* • 30 分钟
  打开 ccboard 费用分析标签页

- **[creational](skills/design-patterns/reference/creational.md)** *中级* • 30 分钟
  单例、工厂、构建器、原型等对象创建模式参考

- **[cyber-defense-team](skills/cyber-defense-team/SKILL.md)** *中级* • 30 分钟
  编排 4 智能体网络防御流水线以分析日志文件中的威胁。适合调查安全日志、检测访问模式异常、评估入侵严重程度或从 nginx/auth/syslog 文件生成事件报告时使用。

- **[dashboard](skills/ccboard/commands/dashboard.md)** *中级* • 30 分钟
  启动 ccboard TUI 仪表盘

- **[design-patterns](skills/design-patterns/SKILL.md)** *中级* • 30 分钟
  检测、建议和评估 TypeScript/JavaScript 代码库中的 GoF 设计模式。适合重构代码、应用单例/工厂/观察者/策略模式、审查模式质量或为 React、Angular、NestJS 和 Vue 寻找栈原生替代方案时使用。

- **[eval-rules](skills/eval-rules/SKILL.md)** *中级* • 30 分钟
  审计 .claude/rules/ 文件的结构正确性、glob 有效性和实际可用性。将每个路径模式与实际项目文件解析匹配，然后询问用户每条规则是否仍然相关且有用。可根据回答就地更新规则。适合首次配置规则、调试触发过于频繁或从不触发的规则，或进行定期规则清理时使用。

- **[eval-skills](skills/eval-skills/SKILL.md)** *中级* • 30 分钟
  审计当前项目中所有技能的 frontmatter 完整性、effort 级别适当性、allowed-tools 范围和内容质量。为每个技能生成含 effort 级别建议的评分报告。适合入职新项目、发布前审查技能质量或为现有技能库添加 effort 字段时使用。

- **[feedback-draft](skills/talk-pipeline/stage-4-position/templates/feedback-draft.md)** *中级* • 30 分钟
  用法：在提交 CFP 或最终确定脚本之前发送给 1-2 位可信同行。

- **[git-ai-archaeology](skills/git-ai-archaeology/SKILL.md)** *中级* • 30 分钟
  分析 Git 仓库中的 AI 配置演进 — 每个路径的首次提交、月度分布、重大 PR 和成熟阶段

- **[guide-recap](skills/guide-recap/SKILL.md)** *中级* • 30 分钟
  将 CHANGELOG 条目转换为社交内容（LinkedIn、Twitter/X、Newsletter、Slack），支持中英双语。适合发版后或每周从指南更新生成版本说明、公告、社交媒体帖子或摘要时使用。

- **[issue-comment](skills/issue-triage/templates/issue-comment.md)** *中级* • 30 分钟
  在 `/issue-triage` 第 3 阶段使用这些模板生成 GitHub issue 评论。评论经过审核

- **[issue-triage](skills/issue-triage/SKILL.md)** *中级* • 30 分钟
  3 阶段 issue 积压管理，包含审计、深度分析和经验证的分类操作。适合分类 GitHub issue、整理 bug 报告、清理过期工单或检测重复 issue 时使用。参数：'all' 分析全部，issue 编号聚焦特定 issue（如 '42 57'），'en'/'fr' 指定语言，无参数 = 仅审计。

- **[kimi-prompt-template](skills/talk-pipeline/stage-5-script/templates/kimi-prompt-template.md)** *中级* • 30 分钟
  > 将此完整提示复制粘贴到 Kimi.com 以生成演示文稿。

- **[landing-page-generator](skills/landing-page-generator/SKILL.md)** *中级* • 30 分钟
  从任何仓库生成完整的、可直接部署的落地页。适合为开源项目创建主页、构建项目网站、将 README 转换为营销页面或在多个仓库间统一落地页风格时使用。

- **[landing-pattern](skills/landing-page-generator/references/landing-pattern.md)** *中级* • 30 分钟
  `claude-code-ultimate-guide-landing` 中已建立的落地页模式文档

- **[linkedin-template](skills/guide-recap/assets/linkedin-template.md)** *中级* • 30 分钟
  目标：约 1300 个字符。结构：钩子 + 上下文 + 要点 + 行动召唤 + 话题标签。

- **[mcp-integration-reference](skills/mcp-integration-reference/SKILL.md)** *中级* • 30 分钟
  与 MCP 服务器集成的技能模板。演示参考文件模式：Claude 在进行任何工具调用之前先读取特定领域的 MCP 速查表，从而减少因服务器特定问题导致的查询失败。复制此技能并将 Sentry 示例替换为您的目标 MCP。

- **[mcp-status](skills/ccboard/commands/mcp-status.md)** *中级* • 30 分钟
  打开 ccboard MCP 服务器标签页

- **[newsletter-template](skills/guide-recap/assets/newsletter-template.md)** *中级* • 30 分钟
  目标：约 500 字。具有深度的结构化章节。

- **[pattern-evaluation](skills/design-patterns/checklists/pattern-evaluation.md)** *中级* • 30 分钟
  评估设计模式实现质量的系统性评分标准

- **[pdf-generator](skills/pdf-generator.md)** *中级* • 30 分钟
  使用 Quarto/Typst 技术栈和现代设计模板生成专业 PDF

- **[pr-triage](skills/pr-triage/SKILL.md)** *中级* • 30 分钟
  4 阶段 PR 积压管理，包含审计、深度代码审查、经验证的评论和可选工作树设置。适合分类 Pull Request、跟进待处理代码审查或管理大量开放 PR 积压时使用。参数：'all' 审查全部，PR 编号聚焦特定 PR（如 '42 57'），'en'/'fr' 指定语言，无参数 = 仅审计。

- **[README](skills/ccboard/README.md)** *中级* • 30 分钟
  > 用于监控和管理 Claude Code 的综合 TUI/Web 仪表盘

- **[README](skills/talk-pipeline/README.md)** *中级* • 30 分钟
  6 阶段技能流水线，将原始素材（文章、录音、笔记）转化为完整的演讲内容

- **[README](skills/release-notes-generator/references/README.md)** *中级* • 30 分钟
  本目录包含在技能执行期间按需加载的文档。

- **[README](skills/release-notes-generator/scripts/README.md)** *中级* • 30 分钟
  本目录包含用于确定性、可重复任务的可执行脚本。

- **[README](skills/release-notes-generator/assets/README.md)** *中级* • 30 分钟
  本目录包含模板、图片和样板代码。

- **[release-notes-generator](skills/release-notes-generator/SKILL.md)** *中级* • 30 分钟
  从 Git 提交生成 3 种格式的发版说明（CHANGELOG.md、PR 正文、Slack 公告）。自动分类变更并将技术语言转换为用户友好的表述。适合发版、变更日志、版本说明、新功能汇总或发布公告时使用。

- **[review-comment](skills/pr-triage/templates/review-comment.md)** *中级* • 30 分钟
  使用此模板生成 GitHub PR 审查评论。根据代码审查结果填写每个部分

- **[rtk-optimizer](skills/rtk-optimizer/SKILL.md)** *中级* • 30 分钟
  用 RTK 包装高输出量 Shell 命令以减少 token 消耗。适合运行 git log、git diff、cargo test、pytest 或其他浪费上下文窗口 token 的冗长 CLI 输出时使用。

- **[security-checklist](skills/security-checklist.md)** *中级* • 30 分钟
  Web 应用的全面安全检查清单

- **[sentry-mcp](skills/mcp-integration-reference/references/sentry-mcp.md)** *中级* • 30 分钟
  Sentry MCP 服务器的参考文件。在进行任何 Sentry MCP 调用前阅读本文件，其中包含

- **[sessions](skills/ccboard/commands/sessions.md)** *中级* • 30 分钟
  浏览 Claude Code 会话历史

- **[skill-creator](skills/skill-creator/SKILL.md)** *中级* • 30 分钟
  构建含 SKILL.md、frontmatter 和捆绑资源的新 Claude Code 技能脚手架。适合创建自定义技能、跨团队统一技能结构或将技能打包分发时使用。

- **[slack-template](skills/guide-recap/assets/slack-template.md)** *中级* • 30 分钟
  简洁、易扫描、富含表情符号。可直接粘贴使用。

- **[slack-template](skills/release-notes-generator/assets/slack-template.md)** *中级* • 30 分钟
  用于生成以产品为中心的 Slack 消息的模板。

- **[smart-explore](skills/smart-explore.md)** *中级* • 30 分钟
  使用 tree-sitter AST 进行渐进式代码探索 — 先获取结构，再深入细节。将每个文件的代码阅读从 10-15k token 减少到 200-500 token。

- **[structural](skills/design-patterns/reference/structural.md)** *中级* • 30 分钟
  适配器、装饰器、外观、代理等组合模式参考

- **[talk-pipeline](skills/talk-pipeline/orchestrator/SKILL.md)** *中级* • 30 分钟
  编排从原始素材到修订表的完整演讲准备流水线，按顺序运行 6 个阶段，在 REX 或概念模式演讲中设置人机协作检查点。适合启动新演讲流水线、从特定阶段恢复流水线或运行完整端到端准备工作流时使用。

- **[talk-stage1-extract](skills/talk-pipeline/stage-1-extract/SKILL.md)** *中级* • 30 分钟
  从源素材（文章、录音、笔记）中提取和整理内容，生成含叙事弧、主题、指标和缺口的演讲摘要。自动检测 REX 与概念类型。适合从任何源素材开始新演讲或在确定演讲前审查现有素材时使用。

- **[talk-stage2-research](skills/talk-pipeline/stage-2-research/SKILL.md)** *中级* • 30 分钟
  执行 Git 考古、变更日志分析，并通过交叉引用 Git 历史与源素材来构建经验证的事实时间线。仅限 REX 模式 — 概念模式下自动跳过。适合构建 REX 演讲且需要从 Git 仓库获取经验证的提交指标、发版时间线和贡献者数据时使用。

- **[talk-stage3-concepts](skills/talk-pipeline/stage-3-concepts/SKILL.md)** *中级* • 30 分钟
  从演讲摘要和时间线构建编号、分类的概念目录，为每个概念评定演讲潜力 HIGH/MEDIUM/LOW，并可选择仓库扩充。适合在选择演讲角度之前需要结构化概念清单，或评估哪些想法具有最强演示潜力时使用。

- **[talk-stage4-position](skills/talk-pipeline/stage-4-position/SKILL.md)** *中级* • 30 分钟
  生成 3-4 个战略性演讲角度，包含优劣势分析、标题选项、CFP 描述和同行反馈草稿，然后强制执行必要的检查点以确认用户意见，再进行脚本编写。适合决定演讲框架、准备 CFP 提交或在多个叙事角度中做选择时使用。

- **[talk-stage5-script](skills/talk-pipeline/stage-5-script/SKILL.md)** *中级* • 30 分钟
  生成完整的 5 幕演讲提纲（含演讲者备注）、逐张幻灯片规格说明，以及用于 AI 幻灯片生成的即用 Kimi 提示。需要第 4 阶段确认的角度和标题。适合已确定演讲角度并需要完整脚本、幻灯片规格和 AI 生成演示文稿提示时使用。

- **[talk-stage6-revision](skills/talk-pipeline/stage-6-revision/SKILL.md)** *中级* • 30 分钟
  生成按幕快速导航的修订表、主概念-URL 对照表、含 6-10 个预期问题的 Q&A 速查表、术语表和外部资源列表。适合准备含 Q&A 的演讲、为听众创建可分享参考资料或为现场演讲构建安全备忘术语表时使用。

- **[tdd-workflow](skills/tdd-workflow.md)** *中级* • 30 分钟
  测试驱动开发工作流与最佳实践

- **[tech-to-product-mappings](skills/release-notes-generator/references/tech-to-product-mappings.md)** *中级* • 30 分钟
  本文档定义如何将技术性提交信息转换为用户友好的产品语言

- **[token-audit](skills/token-audit/skill.md)** *中级* • 30 分钟
  审计 Claude Code 配置以测量固定上下文 token 开销，并生成优先级行动计划

- **[tone-guidelines](skills/guide-recap/references/tone-guidelines.md)** *中级* • 30 分钟
  从 CHANGELOG 条目生成社交内容的规则。核心原则：**通过价值驱动参与**

- **[twitter-template](skills/guide-recap/assets/twitter-template.md)** *中级* • 30 分钟
  两种模式：单条推文（280 字符）或话题串（2-3 条推文）。

- **[version-output](skills/guide-recap/examples/version-output.md)** *中级* • 30 分钟
  输入：`/guide-recap v3.20.5`

- **[voice-refine](skills/voice-refine/SKILL.md)** *中级* • 30 分钟
  将冗长的语音输入转换为结构化、token 高效的 Claude 提示。适合清理包含填充词、重复和无结构思路的语音备忘录、口述内容或语音转文字稿时使用。

- **[week-output](skills/guide-recap/examples/week-output.md)** *中级* • 30 分钟
  输入：`/guide-recap week 2026-01-27`


### 钩子（37）

- **[auto-checkpoint](hooks/bash/auto-checkpoint.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[auto-format](hooks/bash/auto-format.sh)** *中级* • 30 分钟
  INPUT=$(cat)

- **[auto-format](hooks/powershell/auto-format.ps1)** *中级* • 30 分钟
  $inputJson = [Console]::In.ReadToEnd() | ConvertFrom-Json

- **[auto-rename-session](hooks/bash/auto-rename-session.sh)** *中级* • 30 分钟
  set -uo pipefail

- **[claudemd-scanner](hooks/bash/claudemd-scanner.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[dangerous-actions-blocker](hooks/bash/dangerous-actions-blocker.sh)** *中级* • 30 分钟
  set -e

- **[file-guard](hooks/bash/file-guard.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[governance-enforcement-hook](hooks/bash/governance-enforcement-hook.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[identity-reinjection](hooks/bash/identity-reinjection.sh)** *中级* • 30 分钟
  set -uo pipefail

- **[learning-capture](hooks/bash/learning-capture.sh)** *中级* • 30 分钟
  set -e

- **[mcp-config-integrity](hooks/bash/mcp-config-integrity.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[notification](hooks/bash/notification.sh)** *中级* • 30 分钟
  set -e

- **[output-secrets-scanner](hooks/bash/output-secrets-scanner.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[output-validator](hooks/bash/output-validator.sh)** *中级* • 30 分钟
  set -e

- **[permission-request](hooks/bash/permission-request.sh)** *中级* • 30 分钟
  INPUT=$(cat)

- **[pre-commit-evaluator](hooks/bash/pre-commit-evaluator.sh)** *中级* • 30 分钟
  set -e

- **[pre-commit-secrets](hooks/bash/pre-commit-secrets.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[privacy-warning](hooks/bash/privacy-warning.sh)** *中级* • 30 分钟
  if [[ -n "$PRIVACY_WARNING_SHOWN" ]]; then

- **[prompt-injection-detector](hooks/bash/prompt-injection-detector.sh)** *中级* • 30 分钟
  set -e

- **[repo-integrity-scanner](hooks/bash/repo-integrity-scanner.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[rtk-auto-wrapper](hooks/bash/rtk-auto-wrapper.sh)** *中级* • 30 分钟
  if ! command -v rtk &> /dev/null; then

- **[rtk-baseline](hooks/bash/rtk-baseline.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[sandbox-validation](hooks/bash/sandbox-validation.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[security-check](hooks/bash/security-check.sh)** *中级* • 30 分钟
  INPUT=$(cat)

- **[security-check](hooks/powershell/security-check.ps1)** *中级* • 30 分钟
  $inputJson = [Console]::In.ReadToEnd() | ConvertFrom-Json

- **[security-gate](hooks/bash/security-gate.sh)** *中级* • 30 分钟
  set -e

- **[session-logger](hooks/bash/session-logger.sh)** *中级* • 30 分钟
  set -e

- **[session-summary](hooks/bash/session-summary.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[session-summary-config](hooks/bash/session-summary-config.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[setup-init](hooks/bash/setup-init.sh)** *中级* • 30 分钟
  INPUT=$(cat)

- **[smart-suggest](hooks/bash/smart-suggest.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[subagent-stop](hooks/bash/subagent-stop.sh)** *中级* • 30 分钟
  INPUT=$(cat)

- **[test-on-change](hooks/bash/test-on-change.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[tts-selective](hooks/bash/tts-selective.sh)** *中级* • 30 分钟
  set -e

- **[typecheck-on-save](hooks/bash/typecheck-on-save.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[unicode-injection-scanner](hooks/bash/unicode-injection-scanner.sh)** *中级* • 30 分钟
  set -euo pipefail

- **[velocity-governor](hooks/bash/velocity-governor.sh)** *中级* • 30 分钟
  set -euo pipefail


### 工作流（3）

- **[database-branch-setup](workflows/database-branch-setup.md)** *中级* • 30 分钟
  使用 Neon 或 PlanetScale 数据库分支进行隔离功能开发的指南

- **[memory-stack-integration](workflows/memory-stack-integration.md)** *中级* • 30 分钟
  结合 claude-mem、Serena、grepai 和 rg 进行认证重构的 5 天冲刺示例

- **[remotion-quickstart](workflows/remotion-quickstart.md)** *中级* • 30 分钟
  使用 Remotion 和 Claude Code 创建程序化视频的 15 分钟快速入门


### 脚本（2）

- **[ai-usage-charter-template](scripts/ai-usage-charter-template.md)** *中级* • 30 分钟
  > **模板** — 复制到您组织文档仓库中的 `docs/ai-usage-charter.md`。

- **[README](scripts/README.md)** *中级* • 30 分钟
  面向 Claude Code 高级用户的实用脚本：审计、健康检查和会话管理


---

## 按领域浏览

### 通用（181）

- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **README** （中级，30 分钟）
- **adr-writer** （中级，30 分钟）
- **ai-usage-charter-template** （中级，30 分钟）
- **analytics-agent** （中级，30 分钟）
- **anomaly-detector** （中级，30 分钟）
- **architecture-reviewer** （中级，30 分钟）
- **ast-grep-patterns** （中级，30 分钟）
- **audit-agents-skills** （中级，30 分钟）
- **audit-agents-skills** （中级，30 分钟）
- **audit-codebase** （中级，30 分钟）
- **auto-checkpoint** （中级，30 分钟）
- **auto-format** （中级，30 分钟）
- **auto-format** （中级，30 分钟）
- **auto-rename-session** （中级，30 分钟）
- **autoresearch** （中级，30 分钟）
- **before-after** （中级，30 分钟）
- **behavioral** （中级，30 分钟）
- **canary** （中级，30 分钟）
- **catchup** （中级，30 分钟）
- **ccboard** （中级，30 分钟）
- **ccboard-install** （中级，30 分钟）
- **ccboard-web** （中级，30 分钟）
- **changelog-parsing-rules** （中级，30 分钟）
- **changelog-template** （中级，30 分钟）
- **check-cache-bugs** （中级，30 分钟）
- **ci:all** （中级，30 分钟）
- **ci:pipeline** （中级，30 分钟）
- **ci:status** （中级，30 分钟）
- **ci:tests** （中级，30 分钟）
- **claudemd-scanner** （中级，30 分钟）
- **code-reviewer** （中级，30 分钟）
- **commit** （中级，30 分钟）
- **commit-categories** （中级，30 分钟）
- **content-transformation** （中级，30 分钟）
- **costs** （中级，30 分钟）
- **create-handoff** （中级，30 分钟）
- **creational** （中级，30 分钟）
- **cyber-defense-team** （中级，30 分钟）
- **dangerous-actions-blocker** （中级，30 分钟）
- **dashboard** （中级，30 分钟）
- **database-branch-setup** （中级，30 分钟）
- **design-patterns** （中级，30 分钟）
- **devops-sre** （中级，30 分钟）
- **diagnose** （中级，30 分钟）
- **eval-rules** （中级，30 分钟）
- **eval-skills** （中级，30 分钟）
- **explain** （中级，30 分钟）
- **feedback-draft** （中级，30 分钟）
- **file-guard** （中级，30 分钟）
- **generate-tests** （中级，30 分钟）
- **git-ai-archaeology** （中级，30 分钟）
- **git-worktree** （中级，30 分钟）
- **git-worktree-clean** （中级，30 分钟）
- **git-worktree-remove** （中级，30 分钟）
- **git-worktree-status** （中级，30 分钟）
- **governance-enforcement-hook** （中级，30 分钟）
- **guide-recap** （中级，30 分钟）
- **identity-reinjection** （中级，30 分钟）
- **implementer** （中级，30 分钟）
- **integration-reviewer** （中级，30 分钟）
- **investigate** （中级，30 分钟）
- **issue-comment** （中级，30 分钟）
- **issue-triage** （中级，30 分钟）
- **kimi-prompt-template** （中级，30 分钟）
- **land-and-deploy** （中级，30 分钟）
- **landing-page-generator** （中级，30 分钟）
- **landing-pattern** （中级，30 分钟）
- **learn-alternatives** （中级，30 分钟）
- **learn-quiz** （中级，30 分钟）
- **learn-teach** （中级，30 分钟）
- **learning-capture** （中级，30 分钟）
- **linkedin-template** （中级，30 分钟）
- **log-ingestor** （中级，30 分钟）
- **loop-monitor** （中级，30 分钟）
- **mcp-config-integrity** （中级，30 分钟）
- **mcp-integration-reference** （中级，30 分钟）
- **mcp-status** （中级，30 分钟）
- **memory-stack-integration** （中级，30 分钟）
- **methodology-advisor** （中级，30 分钟）
- **newsletter-template** （中级，30 分钟）
- **notification** （中级，30 分钟）
- **optimize** （中级，30 分钟）
- **output-evaluator** （中级，30 分钟）
- **output-secrets-scanner** （中级，30 分钟）
- **output-validator** （中级，30 分钟）
- **pattern-evaluation** （中级，30 分钟）
- **pdf-generator** （中级，30 分钟）
- **permission-request** （中级，30 分钟）
- **plan-ceo-review** （中级，30 分钟）
- **plan-challenger** （中级，30 分钟）
- **plan-eng-review** （中级，30 分钟）
- **plan-execute** （中级，30 分钟）
- **plan-start** （中级，30 分钟）
- **plan-validate** （中级，30 分钟）
- **planner** （中级，30 分钟）
- **planning-coordinator** （中级，30 分钟）
- **pr** （中级，30 分钟）
- **pr-triage** （中级，30 分钟）
- **pre-commit-evaluator** （中级，30 分钟）
- **pre-commit-secrets** （中级，30 分钟）
- **privacy-warning** （中级，30 分钟）
- **prompt-injection-detector** （中级，30 分钟）
- **qa** （中级，30 分钟）
- **recipe-template** （中级，30 分钟）
- **refactor** （中级，30 分钟）
- **refactoring-specialist** （中级，30 分钟）
- **release-notes** （中级，30 分钟）
- **release-notes-generator** （中级，30 分钟）
- **remotion-quickstart** （中级，30 分钟）
- **repo-integrity-scanner** （中级，30 分钟）
- **report-template** （中级，30 分钟）
- **resume-handoff** （中级，30 分钟）
- **review-comment** （中级，30 分钟）
- **review-plan** （中级，30 分钟）
- **review-pr** （中级，30 分钟）
- **risk-classifier** （中级，30 分钟）
- **routines-discover** （中级，30 分钟）
- **rtk-auto-wrapper** （中级，30 分钟）
- **rtk-baseline** （中级，30 分钟）
- **rtk-optimizer** （中级，30 分钟）
- **sandbox-status** （中级，30 分钟）
- **sandbox-validation** （中级，30 分钟）
- **scaffold** （中级，30 分钟）
- **security** （中级，30 分钟）
- **security-audit** （中级，30 分钟）
- **security-auditor** （中级，30 分钟）
- **security-check** （中级，30 分钟）
- **security-check** （中级，30 分钟）
- **security-check** （中级，30 分钟）
- **security-checklist** （中级，30 分钟）
- **security-gate** （中级，30 分钟）
- **security-patcher** （中级，30 分钟）
- **sentry-mcp** （中级，30 分钟）
- **session-logger** （中级，30 分钟）
- **session-save** （中级，30 分钟）
- **session-summary** （中级，30 分钟）
- **session-summary-config** （中级，30 分钟）
- **sessions** （中级，30 分钟）
- **setup-init** （中级，30 分钟）
- **ship** （中级，30 分钟）
- **skill-creator** （中级，30 分钟）
- **slack-template** （中级，30 分钟）
- **slack-template** （中级，30 分钟）
- **smart-explore** （中级，30 分钟）
- **smart-suggest** （中级，30 分钟）
- **sonarqube** （中级，30 分钟）
- **structural** （中级，30 分钟）
- **subagent-stop** （中级，30 分钟）
- **talk-pipeline** （中级，30 分钟）
- **talk-stage1-extract** （中级，30 分钟）
- **talk-stage2-research** （中级，30 分钟）
- **talk-stage3-concepts** （中级，30 分钟）
- **talk-stage4-position** （中级，30 分钟）
- **talk-stage5-script** （中级，30 分钟）
- **talk-stage6-revision** （中级，30 分钟）
- **tdd-workflow** （中级，30 分钟）
- **tech-to-product-mappings** （中级，30 分钟）
- **test-on-change** （中级，30 分钟）
- **test-writer** （中级，30 分钟）
- **threat-reporter** （中级，30 分钟）
- **token-audit** （中级，30 分钟）
- **tone-guidelines** （中级，30 分钟）
- **tts-selective** （中级，30 分钟）
- **twitter-template** （中级，30 分钟）
- **typecheck-on-save** （中级，30 分钟）
- **unicode-injection-scanner** （中级，30 分钟）
- **update-handoff** （中级，30 分钟）
- **update-threat-db** （中级，30 分钟）
- **validate-changes** （中级，30 分钟）
- **velocity-governor** （中级，30 分钟）
- **version-output** （中级，30 分钟）
- **voice-refine** （中级，30 分钟）
- **week-output** （中级，30 分钟）

---

## 初学者入门

推荐首次使用者参考的模板：

暂无明确标注为初学者友好的模板。

---

## 元数据参考

模板可在 YAML frontmatter 中包含以下元数据：

```yaml

name: template-name

description: What this template does

complexity: beginner|intermediate|advanced

time: 5 min|15 min|30 min|1 hour|2 hours|4+ hours|varies

domain: security|testing|deployment|general|...

prerequisites: [skill1, skill2]

status: stable|experimental|deprecated

keywords: [tag1, tag2]

```
