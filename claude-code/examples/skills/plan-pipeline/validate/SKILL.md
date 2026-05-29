> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: plan-pipeline-validate
description: "双层计划验证：即时结构检查 + 触发式专家智能体。使用 ADR 和第一性原理自动修复问题。所有问题必须在执行前解决。"
effort: medium
disable-model-invocation: true
---

# /plan-pipeline:validate — 双层验证

独立验证 `/plan-pipeline:start` 生成的计划。不编写任何代码。运行此命令后，在执行 `/plan-pipeline:execute` 前先运行 `/clear`。

验证与规划刻意分离：没有参与编写计划的验证者不会受其假设的锚定效应影响。

---

## 前提条件

必须存在已提交的计划文件，路径为 `docs/plans/plan-{name}.md`。若存在多个计划，列出后询问用户要验证哪个。

---

## 第一层：结构验证

立即执行，无需智能体。检查计划文档是否满足：

**格式与完整性**
- [ ] 所有必需章节均存在（摘要、决策、架构、任务、测试计划、范围外）
- [ ] 每个任务包含：描述、受影响文件、验收标准、层级分配

**依赖链**
- [ ] 任务之间无循环依赖
- [ ] 高层任务仅依赖低层任务
- [ ] 所有声明的依赖在计划中均存在

**文件存在性**
- [ ] 列出的每个待修改文件在代码库中确实存在（使用 Glob 检查）
- [ ] 新文件按项目规范放置在合适目录中

**ADR 一致性**
- [ ] 计划决策与 `/plan-pipeline:start` 期间创建的 ADR 一致
- [ ] 与 `docs/adr/` 中现有 ADR 无矛盾

**CLAUDE.md 合规性**
- [ ] 计划遵守 CLAUDE.md 中的所有硬性规则
- [ ] 无第一性原理违规（无变通方案、无向后兼容垫片）

**测试覆盖**
- [ ] 每个新函数/组件均有对应的测试任务
- [ ] 标记为 TDD 的任务，其失败测试编写在实现任务之前

在进入第二层前，记录所有第一层问题并标注严重程度（BLOCKER / WARNING / INFO）。

---

## 第二层：专家评审

通过将触发规则应用于计划内容来选择智能体。无需用户输入——触发条件是客观的。

**验证智能体池：**

| 智能体 | 触发条件 | 模型 |
|-------|---------|-------|
| `security-reviewer` | 认证、支付、PII、RBAC、新公开 API | Opus |
| `db-migration-reviewer` | 新表、列、索引或迁移文件 | Opus |
| `performance-reviewer` | 新查询、解析器、路由或新增依赖 | Sonnet |
| `design-system-reviewer` | 新 UI 组件或视觉样式变更 | Sonnet |
| `ux-reviewer` | 新页面、表单、弹窗或交互模式 | Sonnet |
| `cross-platform-reviewer` | 同时涉及 Web 和移动端或共享包的变更 | Sonnet |
| `native-app-reviewer` | 移动端页面、原生 UI 包变更 | Sonnet |
| `integration-reviewer` | 新外部服务、库或 OTEL 配置 | Opus |

并行启动触发的智能体（Task 工具，run_in_background: true）。每个智能体接收：计划文件、相关 ADR，以及基于其领域的针对性问题。

通过 `Read` 监控 `.claude/tasks/<id>/output.log`（TaskOutput 自 v2.1.83 起已弃用）。向用户汇报进度。

每个智能体必须返回结构化发现：
```
FINDING: [BLOCKER|WARNING|INFO]
Location: [plan section or file reference]
Issue: [concrete description]
Risk: [what breaks if this isn't addressed]
Suggestion: [specific fix or alternative]
```

---

## 自动修复阶段

将第一层结构问题与第二层专家发现合并为单一问题列表。每个问题必须解决，不可跳过。

**对每个问题进行分类：**

**A 桶 —— 自动解决：**
- 问题符合现有 ADR 决策 → 引用 ADR，标记为已解决
- 问题符合 PATTERNS.md 中已确认的模式 → 引用模式，标记为已解决
- 问题可通过 CLAUDE.md 中的第一性原理解决 → 应用规则，标记为已解决

**B 桶 —— 需要人工输入：**
- 现有决策未覆盖的新架构问题
- 无明确先例的 ADR 冲突
- 无明显解决方案的阻塞问题

对于 B 桶问题：呈现问题，说明无法自动解决的原因，提出选项，等待决策。将决策记录到计划的 `## 决策` 章节，若具有架构意义则创建新 ADR。

**所有问题分类完成后批量应用修复。** 更新计划文件并提交。

---

## 问题持久化

将每个问题记录到 `docs/plans/metrics/{name}.json` 的 `validation.issues` 下：

```json
{
  "id": "S-001",
  "layer": 1,
  "severity": "WARNING",
  "category": "test-coverage",
  "description": "No test task for the new webhook handler",
  "reporting_agent": "structural",
  "triage": "A",
  "resolution_source": "first-principles",
  "resolution": "Added test task in Layer 2 of the plan"
}
```

---

## 自动跳转

若所有问题均自动解决（仅 A 桶）：无需询问，自动启动 `/plan-pipeline:execute`。

若有任何问题需要人工输入（B 桶）：询问"所有问题已解决。准备执行吗？"后再继续。

---

## 使用方式

```
/plan-pipeline:validate
```

自动拾取最近未提交的计划。或指定计划：

```
/plan-pipeline:validate plan-user-authentication
```

## 输出

```
Layer 1: Structural validation...
  ✓ Format complete
  ✓ Dependencies valid
  ⚠ WARNING S-001: Missing test task for webhook handler
  ✓ CLAUDE.md compliant

Layer 2: Triggering specialist agents...
  → security-reviewer (auth changes detected) [Opus]
  → db-migration-reviewer (new users table) [Opus]
  → performance-reviewer (new query in /api/users) [Sonnet]
  Monitoring... 1/3 complete... 2/3 complete... done.

  BLOCKER B-001 [security-reviewer]: JWT expiry not validated on refresh endpoint
  WARNING B-002 [db-migration-reviewer]: Migration lacks rollback strategy

Auto-fix phase:
  S-001 → auto-resolved (first principles: test coverage rule)
  B-001 → NEEDS INPUT (no existing ADR for JWT refresh strategy)
  B-002 → auto-resolved (ADR-0003: migration rollback pattern)

[User input requested for B-001]
Decision recorded. ADR-0011 created.

All 3 issues resolved. Plan updated.
→ Auto-starting /plan-pipeline:execute
```

## 何时使用

始终使用——在任何 `/plan-pipeline:execute` 调用之前。验证成本（$0.20-3.00）相比在执行途中发现问题的代价微不足道。

## 流水线位置

```
/plan-pipeline:ceo-review    → 产品方向锁定
/plan-pipeline:eng-review    → 架构锁定
/plan-pipeline:start         → 生成实现计划
/plan-pipeline:validate      → 执行前验证     ← 当前位置
/plan-pipeline:execute       → 执行至 PR 合并
```
