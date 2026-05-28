> 📚 **AI Spark Wiki** · Claude Code 知识库

# Claude Code 知识库

> 基于 [claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) v3.41.0 整理，结合官方文档 [code.claude.com](https://code.claude.com)。

## 快速入口

| 我想要... | 去哪里 |
|-----------|--------|
| 快速上手 | [学习路径 Module 1](./learning-path/01-installation.md) |
| 一页速查 | [Cheatsheet](./cheatsheet.md) |
| 完整系统学习 | [Ultimate Guide](./ultimate-guide.md) |
| 理解架构原理 | [架构深度文档](./core/architecture.md) |
| 配置记忆系统 | [Memory Systems](./core/memory-systems.md) |
| 上下文工程 | [Context Engineering](./core/context-engineering.md) |
| 安全加固 | [Security Hardening](./security/security-hardening.md) |
| 工作流模板 | [Workflows](./workflows/) |
| 可视化图表 | [Diagrams](./diagrams/) |

## 目录结构

```
claude-code/
├── ultimate-guide.md          # 主文档（2.4万行，完整指南）
├── cheatsheet.md              # 速查表（日常必备命令）
│
├── learning-path/             # 7模块结构化学习路径（8-11小时）
│   ├── 01-installation.md
│   ├── 02-core-loop.md
│   ├── 03-memory.md
│   ├── 04-agents.md
│   ├── 05-skills.md
│   ├── 06-hooks.md
│   └── 07-advanced.md
│
├── core/                      # 核心深度文档
│   ├── architecture.md        # 内部架构分析
│   ├── context-engineering.md # 上下文工程
│   ├── memory-systems.md      # 记忆层级系统
│   ├── methodologies.md       # TDD/SDD/BDD 工作流
│   ├── settings-reference.md  # 完整配置参考
│   ├── skill-design-patterns.md
│   ├── agent-harness.md
│   ├── glossary.md
│   ├── known-issues.md
│   ├── claude-code-releases.md
│   └── visual-reference.md
│
├── workflows/                 # 具体工作流（25个）
│   ├── tdd-with-claude.md
│   ├── code-review.md
│   ├── agent-teams.md
│   ├── github-actions.md
│   └── ...
│
├── diagrams/                  # Mermaid 可视化图表（12组）
│   ├── 01-foundations.md
│   ├── 04-architecture-internals.md
│   ├── 07-multi-agent-patterns.md
│   └── ...
│
├── security/                  # 安全文档
│   ├── security-hardening.md  # 28个 CVE + 威胁数据库
│   ├── data-privacy.md
│   ├── enterprise-governance.md
│   └── production-safety.md
│
├── ecosystem/                 # 生态工具
│   ├── mcp-servers-ecosystem.md
│   ├── agentic-tools.md
│   └── ...
│
├── ops/                       # 运维/团队指标
├── roles/                     # 角色与职业路径
├── examples/                  # 181个生产级模板
└── quiz/                      # 271题知识测验
```

## 推荐学习顺序

**新手**（第一周）：
1. `learning-path/01` → `02` → `03` — 装好、跑起来、理解记忆
2. `cheatsheet.md` — 贴在桌面
3. `ultimate-guide.md` 第1-3章

**进阶**（第二周）：
4. `learning-path/04-agents.md` + `05-skills.md`
5. `core/context-engineering.md`
6. `workflows/` 选和自己工作相关的

**深入**：
7. `core/architecture.md` — 理解内部机制
8. `security/security-hardening.md`
9. `diagrams/` 系列 — 可视化理解各子系统
