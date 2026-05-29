> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: methodology-advisor
description: 分析您的代码库并提出 3 个针对性问题，从而推荐最适合的 AI 辅助开发方法论组合
effort: medium
---

# 方法论顾问

分析此项目并推荐最佳的 AI 辅助开发方法论组合。先尽量从代码库中读取信息，再仅询问无法推断的内容。

**耗时**：2-4 分钟 | **输出**：一个推荐的方法论组合 + 定制化快速入门指南

---

## 第 1 阶段 — 静默代码库分析

静默执行以下读取操作。暂不输出结果——仅建立内部分析图景。

### 1.1 项目标识

```bash
# 配置文件
cat CLAUDE.md 2>/dev/null || cat claude.md 2>/dev/null
cat package.json 2>/dev/null | grep -E '"name"|"description"|"scripts"' | head -10
cat Cargo.toml 2>/dev/null | grep -E '^name|^description' | head -5
cat pyproject.toml 2>/dev/null | grep -E '^name|^description' | head -5
cat go.mod 2>/dev/null | head -3
```

### 1.2 团队规模

```bash
# 过去 90 天的独立贡献者数
git log --since="90 days ago" --format="%ae" 2>/dev/null | sort -u | wc -l
# 总提交数
git log --oneline 2>/dev/null | wc -l
```

### 1.3 测试成熟度

```bash
# 测试文件是否存在？
find . -name "*.test.*" -o -name "*.spec.*" -o -name "*_test.*" -o -name "test_*.py" \
  2>/dev/null | grep -v node_modules | grep -v ".git" | wc -l
# 测试框架线索
grep -rn --include="*.json" --include="*.toml" --include="*.yaml" \
  -l "jest\|vitest\|pytest\|rspec\|mocha\|cypress\|playwright" \
  2>/dev/null | grep -v node_modules | head -5
# CI 配置
ls .github/workflows/*.yml 2>/dev/null | wc -l
ls .gitlab-ci.yml .circleci/config.yml 2>/dev/null | wc -l
```

### 1.4 规格说明与文档信号

```bash
# 规格文件
find . -name "*.spec.md" -o -name "SPEC*.md" -o -name "spec.md" -o -name "DESIGN*.md" \
  -o -name "ADR*.md" -o -name "RFC*.md" \
  2>/dev/null | grep -v node_modules | grep -v ".git" | head -10
# OpenAPI / 契约文件
find . -name "openapi*.yaml" -o -name "openapi*.json" -o -name "swagger*.yaml" \
  -o -name "*.proto" \
  2>/dev/null | grep -v node_modules | head -5
# BDD 功能文件
find . -name "*.feature" 2>/dev/null | grep -v node_modules | wc -l
```

### 1.5 代码库规模与结构

```bash
# 文件数（粗略估计）
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" \
  -o -name "*.rs" -o -name "*.go" -o -name "*.java" -o -name "*.rb" \) \
  2>/dev/null | grep -v node_modules | grep -v ".git" | wc -l
# 服务/包（Monorepo 信号）
ls packages/ apps/ services/ 2>/dev/null | head -10
```

### 1.6 AI 与 LLM 信号

```bash
# 代码中的 LLM API 使用情况
grep -rn --include="*.ts" --include="*.py" --include="*.js" \
  -l "anthropic\|openai\|groq\|mistral\|langchain\|llm\|ChatCompletion\|claude" \
  2>/dev/null | grep -v node_modules | grep -v ".git" | head -5
# 评估框架线索
find . -name "evals*" -o -name "*eval*" -type d 2>/dev/null | grep -v node_modules | head -5
```

---

## 第 2 阶段 — 对 8 个方法论组合评分

根据分析结果，对每个组合按契合度信号评分（0-10）：

| 组合 | 提升评分的关键信号 |
|-------|----------------------------------|
| **solo-mvp** | 1 个贡献者、文件数少、尚无 CI、全新项目 |
| **team-greenfield** | 2-10 个贡献者、新项目、无遗留文件 |
| **microservices** | `packages/`、`services/` 目录、OpenAPI 文件、`.proto` 文件 |
| **brownfield-saas** | 提交数多、文件数多、测试文件少 |
| **enterprise-gov** | 10 个以上贡献者、有 CI、ADR 文件、`AGENTS.md` |
| **llm-native** | LLM 导入、评估目录、AI 产品信号 |
| **power-solo** | 1 个贡献者、提交频率高、迭代式提交 |
| **plan-moderate** | 信号混合、存在 CLAUDE.md、中等规模 |

---

## 第 3 阶段 — 只询问无法推断的内容

静默分析完成后，用 2-3 句话向用户呈现初步判断，然后恰好提问 3 个问题，不多不少。

格式：

```
从您的代码库中我发现：[2-3 条具体观察]。
在给出建议之前，请回答 3 个简短问题：

1. [痛点问题 — 从下方问题库中选择最相关的]
2. [发布频率 — 如果无法从 CI/CD 信号推断]
3. [配置意愿 — 您愿意投入多少前期配置工作？]
```

**问题库 — 根据分析结果选取 3 个最相关的：**

- 痛点："目前最拖慢您进度的是什么——回归问题、需求不清晰、会话间上下文丢失，还是缺乏可追溯性？"
- 痛点："当 Claude 生成大量代码时，您最担心什么——质量、与规格说明的偏差，还是难以追踪构建内容？"
- 发布："您多久向生产环境发布一次——每天多次、每周一次，还是更长的发布周期？"
- 发布："这是一个拥有真实用户的产品、一个原型，还是一个内部工具？"
- 治理："您愿意投入多少初期配置——完全不想（直接开始）、30 分钟，还是半天？"
- 治理："是否有开发团队以外的人员（产品经理、QA、合规）需要验证构建内容？"
- AI 产品："您的产品是否直接向终端用户暴露 AI 生成的输出？"
- 规模："多个服务或团队是否需要在实施前就 API 契约达成一致？"

---

## 第 4 阶段 — 推荐方案

按以下结构输出推荐方案：

---

### 您的方法论组合：[组合名称] [图标]

**为何适合您的项目：**
- [第 1 阶段发现] → [解释此组合选择]
- [第 1 阶段发现] → [解释此组合选择]
- [第 N 个问题的回答] → [解释此组合选择]

**包含的方法论：** `[方法 A]` + `[方法 B]`（+ `[方法 C]`，如适用）

**在实践中的样子：**
[2-3 句话描述针对本项目的具体工作流，使用实际找到的文件名或路径。]

**适合您项目的快速入门：**
1. [结合实际项目上下文的具体第一步]
2. [第二步]
3. [第三步]

**开始前请注意：**
- [该组合的一个客观权衡或局限性]
- [根据分析结果需要关注的一个点]

**深入了解：** https://cc.
**完整方法论指南：** https://cc.

---

## 方法论组合参考（内部）

用于将评分映射到快速入门语言：

**solo-mvp**（SDD + TDD）：在 CLAUDE.md 中编写功能规格 → `"为此规格编写失败测试，然后实现直至测试通过。"`

**team-greenfield**（规格套件 + TDD + BDD）：`/speckit.constitution` → 与产品经理共同编写 Given/When/Then 场景 → TDD 每个场景。

**microservices**（CDD + Specmatic + TDD）：先编写 OpenAPI 规格 → Specmatic 进行契约测试 → TDD 实现。

**brownfield-saas**（OpenSpec + BDD + JiT 测试）：OpenSpec 记录当前状态 → BDD 覆盖变更行为 → 合并前：`"生成能捕获此 diff 中回归问题的测试。"`

**enterprise-gov**（BMAD + 规格套件 + Specmatic）：`constitution.md` → 智能体角色定义 → 规格套件需求 → Specmatic 契约强制执行。

**llm-native**（评估驱动 + 多智能体）：定义评估标准（准确性、安全性、格式）→ 构建评估框架 → 迭代直至评估通过。

**power-solo**（TDD + Ralph 循环 + 迭代式）：紧凑的测试循环 → 通过 git stash + 进度文件在每个任务间保持全新上下文 → `"持续迭代直至所有测试通过且 lint 无误。"`

**plan-moderate**（计划优先 + SDD + 上下文工程）：每个复杂任务先进入规划模式（Shift+Tab）→ 验证 → 在 CLAUDE.md 中编写规格说明 → 通过渐进式上下文加载执行。
