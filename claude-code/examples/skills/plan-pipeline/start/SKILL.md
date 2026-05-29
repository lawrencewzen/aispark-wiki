> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: plan-pipeline-start
description: "5 阶段规划流水线：PRD 分析、设计评审、技术决策、动态调研团队、指标记录。在编写任何代码之前，生成完整的实施计划和架构决策记录（ADR）。"
effort: high
disable-model-invocation: true
---

# /plan-pipeline:start — 5 阶段规划

分析请求并通过结构化阶段生成完整的实施计划。不编写任何代码，所有重要决策均有记录。执行本命令后，运行 `/plan-pipeline:validate` 之前请先执行 `/clear`。

---

## 第 1 阶段：PRD 与设计分析

### 步骤 1.1 — PRD 分析

*如无 PRD（重构、基础设施变更、Bug 修复），跳过此步。*

读取所有 PRD 文件，以及 `docs/INFORMATION_ARCHITECTURE.md`（如存在）。扫描代码库，了解当前实施状态。

将发现归入以下 3 个类别：

**缺失需求** — 缺失或不完整的验收标准
**模糊需求** — 存在多种合理解读的条目
**合规问题** — 安全性、数据隐私、API 合约影响

对于每项发现：提供带有具体利弊权衡的选项，与用户讨论，在继续之前将每项决策记录到计划文件的 `## 决策` 章节中。未解决的模糊问题不得推进。

### 步骤 1.2 — 设计分析

*如无 UI 变更，跳过此步。*

读取：`DESIGN_SYSTEM.md`、现有 UX 架构决策记录、CLAUDE.md 中的 UX 规则。

生成以下规格说明：
- **页面清单**：新增/修改的页面、路由位置、组件复用审计
- **状态目录**：每个交互元素的空、加载中、已填充、错误和部分加载状态
- **交互规格**：用户流程（主路径 + 备选路径）、焦点/键盘行为
- **动画规格**：将每个交互映射到现有关键帧或指定新关键帧，包含 `prefers-reduced-motion` 回退方案
- **响应式行为**：断点、Web/移动端差异决策
- **无障碍性**：WAI-ARIA 模式选择、实时区域、错误可见性

为重要 UX 决策创建设计 ADR（交互模式选择、新动画规范、平台差异）。次要布局决策直接记录到计划文件中。

---

## 第 2 阶段：技术分析

为代码库定向调研生成 1-2 个探索智能体，通过任务工具在后台运行。

智能体运行期间，检查：
- `docs/adr/` 中现有的 ADR —— 若 3 个以上 ADR 均确认同一决策 → 无需询问，自动解决
- PATTERNS.md —— 已确认的模式直接应用

智能体返回后：针对每项架构决策提供 2-3 个选项、具体利弊分析和建议。就每项未解决的决策征求用户意见。

对于每项重要决策：
1. 使用标准 Nygard 格式（上下文 / 决策 / 状态 / 后果）创建 `docs/adr/ADR-XXXX.md`
2. 将新观察结果更新到 `docs/adr/PATTERNS.md`

---

## 第 3 阶段：范围评估

应用触发规则确定需要哪些调研智能体。提出建议团队并说明每个智能体纳入的理由。

**调研智能体池：**

| 智能体 | 触发条件 | 模型 |
|-------|---------|-------|
| `code-explorer` | 始终 | Sonnet |
| `arch-researcher` | 变更涉及 2 个以上架构层 | Sonnet |
| `database-analyst` | 任何数据库 schema 变更 | Sonnet |
| `security-analyst` | 认证、支付、PII、RBAC、限流 | Opus |
| `test-analyzer` | 非简单功能（非仅 Bug 修复） | Sonnet |
| `cross-platform-specialist` | 需要 Web + 移动端一致性 | Sonnet |
| `native-app-specialist` | 任务涉及移动/原生 UI 包 | Sonnet |
| `design-system-researcher` | UI 变更在范围内 | Sonnet |
| `dependency-researcher` | 新增依赖包 | Sonnet |
| `devops-specialist` | Docker、环境变量、CI/CD 变更 | Sonnet |
| `integration-researcher` | 新服务、库、OTEL 配置 | Opus |
| `planning-coordinator` | 始终（当选择 2 个以上智能体时） | Opus |

**级别标签**（描述性，非规定性）：
- Tier 0（0 个智能体）：单独 —— 内联调研，不派生智能体
- Tier 1（1-3 个智能体）：专注型
- Tier 2（4-6 个智能体）：标准型
- Tier 3（7-9 个智能体）：综合型
- Tier 4（10 个以上智能体）：全谱型

告知用户："我建议组建 **[Tier N - 标签]** 团队：[智能体列表，每个附一行理由]。需要添加或移除任何智能体吗？"

等待批准后再进入第 4 阶段。

---

## 第 4 阶段：调研与计划生成

**Tier 0**：内联调研。无需派生智能体，直接编写计划。

**Tier 1+**：使用任务工具并行派生已批准的智能体（run_in_background: true）。对每个智能体提供：
- 其具体调研范围
- 需要调查的相关文件/区域
- 需要回答的问题

通过直接读取其输出文件来监控智能体（TaskOutput 自 v2.1.83 起已弃用 —— 请改用 `Read` 读取 `.claude/tasks/<id>/output.log`）。汇报进度："已完成 3/6 个智能体……"

所有智能体返回后：若已派生 `planning-coordinator`，将所有智能体报告发送给它并让其综合最终计划；否则直接综合。

**计划文件结构**（`docs/plans/plan-{name}.md`）：

```markdown
# 计划：{feature-name}
创建时间：{date} | 分支：{branch-name} | 级别：{N}

## 摘要
一段话：实现内容及原因。

## 决策
第 1 阶段（PRD 分析）中记录的决策。

## 架构
已创建的 ADR、已应用的模式、已做出的架构选择。

## 任务
按层级排列的有序任务列表（1 = 基础层，2 = 依赖层 1，以此类推）

### 第 1 层
- [ ] 任务 A — 描述、涉及文件、验收标准
- [ ] 任务 B — 描述、涉及文件、验收标准

### 第 2 层
- [ ] 任务 C — 依赖 A — 描述、涉及文件、验收标准

## 测试计划
每项任务的验证方式。明确标注 TDD 任务。

## 集成验证
执行后运行的冒烟测试命令（如后端/服务在范围内）。

## 范围之外
本计划明确不涉及的内容。
```

提交：计划文件 + ADR 文件 + 智能体报告清单。

---

## 第 5 阶段：最终指标

将时间戳、各阶段耗时、智能体数量和成本估算记录到 `docs/plans/metrics/{name}.json`。提交。

---

## 自动跳转

若第 1 阶段未产生未解决的模糊问题，且第 2 阶段未产生未解决的决策：自动启动 `/plan-pipeline:validate`，无需询问。

若发生过人工讨论：在继续之前询问"准备好验证此计划了吗？"

---

## 用法

```
/plan-pipeline:start
```

在提示时提供功能描述或指向 PRD 文件。命令会以交互方式处理后续步骤。

## 适用场景

适用于任何非简单功能：涉及 2 个以上文件、包含架构决策、或规划失误代价高昂的情况。

对于简单变更（错别字、简单重构）：改用 `/plan` 模式。

## 流水线位置

```
/plan-pipeline:ceo-review    → 产品方向锁定
/plan-pipeline:eng-review    → 架构方案锁定
/plan-pipeline:start         → 生成实施计划   ← 当前位置
/plan-pipeline:validate      → 执行前验证
/plan-pipeline:execute       → 执行至合并 PR
```
