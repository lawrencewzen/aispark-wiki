> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: plan-pipeline-execute
description: "执行已验证的计划：工作树隔离、TDD 脚手架、基于层级的并行智能体、质量关卡与冒烟测试、PR 创建与合并。处理从执行到 PR 合并的全流程。"
effort: high
disable-model-invocation: true
---

# /plan-pipeline:execute — 执行至 PR 合并

在隔离的工作树中执行已验证的计划。派生每个任务的智能体，验证质量，创建并合并 PR。处理从执行到清理的全流程。

执行此命令前先运行 `/clear`。

---

## 前提条件

`docs/plans/plan-{name}.md` 处必须存在已验证的计划，且所有问题已解决（即 `/plan-pipeline:validate` 的输出）。

---

## 第 1 步：工作树设置

创建隔离的 git 工作树：

```bash
git worktree add .worktrees/{plan-name} -b feature/{plan-name}
```

所有执行均在工作树内进行。主分支在整个过程中保持干净。

---

## 第 2 步：TDD 脚手架

*仅适用于计划中标记为 TDD 的任务。*

对每个 TDD 任务，在任何实现之前：
1. 编写定义验收标准的失败测试
2. 运行测试确认其失败（红灯）
3. 提交失败的测试
4. 在任务中标记测试文件，供实现智能体查找

此步骤不编写实现代码。

---

## 第 3 步：基于层级的并行执行

从计划中解析任务列表，按层分组（第 1 层 = 基础层，第 2 层 = 依赖第 1 层，以此类推）。

**对每一层：**
1. 识别该层中的所有任务
2. 并行为每个任务派生一个智能体（Task 工具，`run_in_background: true`）
3. 每个智能体接收：任务描述、要修改的文件、验收标准以及相关 ADR
4. 通过 `Read` 读取 `.claude/tasks/<id>/output.log` 监控所有智能体（TaskOutput 自 v2.1.83 起已废弃）
5. 每个智能体完成任务时提交：`git commit -m "feat: {task-description}"`
6. 等待该层所有任务完成后再启动下一层

**漂移检测**：每层完成后，将实际变更与计划规格进行 diff 比对。如果实现与计划显著偏离（出现计划外的新文件、计划中的文件未被修改），则标记并询问如何处理。不得在发现漂移后静默继续。

**每个任务的智能体指令：**
```
You are implementing one task from a validated plan.
Task: {description}
Files to modify: {file list}
Acceptance criteria: {criteria}
Relevant ADRs: {adr list}

First principles:
- Build state-of-the-art. No workarounds, no legacy patterns.
- Fix at the correct architectural level, never with component-level hacks.
- If you discover that the plan is wrong or missing context, stop and report — do not improvise architecture.

Commit your changes when complete with message: "feat: {task-description}"
```

---

## 第 4 步：质量关卡

并行运行：
- Linter
- 类型检查器（如适用）
- 完整测试套件

全部通过：继续冒烟测试。

任意失败：派生一个 `quality-fixer` 调试智能体并传入失败输出。最多允许 **3 次自动修复尝试**。每次尝试后重新运行质量关卡。若 3 次尝试后仍失败：停止执行，报告失败信息及完整错误输出，等待人工介入。

**集成冒烟测试** *(纯前端或仅文档的计划可跳过)*：

运行计划 `## Integration Verification` 部分中定义的冒烟命令。此外：
- 若使用 GraphQL：运行内省探测以验证 schema 可访问
- 若使用 Docker 服务：扫描容器日志中的 ERROR 级别条目
- 若有新 API 路由：验证每条路由返回预期状态码

冒烟测试失败由 `quality-fixer-smoke` 智能体调试，同样限 3 次尝试。

---

## 第 5 步：PR 前文档

*在工作树中，创建 PR 之前。*

**PRD 核对**：将实现行为与原始 PRD 对比。记录实现过程中发现的任何偏差或新增内容。用实际情况更新 PRD。这些更新与功能在同一 PR 中交付。

**计划归档**：将 `docs/plans/plan-{name}.md` 移动至 `docs/plans/completed/plan-{name}.md`，并更新状态头部。

提交文档更新：`docs: reconcile PRD and archive plan for {feature-name}`。

---

## 第 6 步：推送与 PR

推送工作树分支并创建 PR：

```bash
git push origin feature/{plan-name}
gh pr create \
  --title "{feature-name}: {one-line summary from plan}" \
  --body "$(cat .pr-body.md)"
```

PR 正文模板：
```markdown
## Summary
{plan summary paragraph}

## Changes
{auto-generated from task list: bullet per task with files affected}

## ADRs
{list of ADRs created during this plan}

## Test Plan
{from plan test plan section}

## Smoke Test Results
{output from integration verification}
```

使用 squash 合并：
```bash
gh pr merge --squash --delete-branch
```

---

## 第 7 步：合并后指标

切回 develop/main。用执行数据更新 `docs/plans/metrics/{name}.json`：
- 任务数量及每层分解
- TDD 任务数量
- Diff 统计（变更文件数、新增/删除行数）
- 质量关卡结果（通过/失败、修复尝试次数）
- 冒烟测试结果
- 漂移评分（0-1，实现与计划的匹配程度）
- PR 数据（编号、合并提交、时间戳）

提交指标更新。

---

## 第 8 步：工作树清理

```bash
git worktree remove .worktrees/{plan-name}
```

---

## 用法

```
/plan-pipeline:execute
```

自动选取最近已验证的计划。或指定计划名称：

```
/plan-pipeline:execute plan-user-authentication
```

## 输出示例

```
Setting up worktree: .worktrees/user-authentication
Branch: feature/user-authentication

TDD scaffolding: 2 tasks marked TDD
  ✓ Written failing tests for: auth-token-validation
  ✓ Written failing tests for: refresh-token-rotation
  Committed: "test: failing tests for auth pipeline (TDD)"

Executing Layer 1 (3 tasks, parallel)...
  [agent-1] Implementing: JWT token generation service
  [agent-2] Implementing: User session model
  [agent-3] Implementing: Auth middleware
  ✓ Layer 1 complete. 3 commits.

Drift check: Layer 1... ✓ No drift detected.

Executing Layer 2 (2 tasks, parallel)...
  [agent-4] Implementing: Login endpoint
  [agent-5] Implementing: Refresh endpoint
  ✓ Layer 2 complete. 2 commits.

Quality gate...
  ✓ Lint passed
  ✓ Type check passed
  ✓ Tests: 47 passed, 0 failed

Smoke test...
  ✓ GraphQL introspection: OK
  ✓ POST /api/auth/login: 200
  ✓ POST /api/auth/refresh: 200

Pre-PR docs...
  ✓ PRD reconciled (1 minor deviation noted)
  ✓ Plan archived to docs/plans/completed/

PR created: #142 "user-authentication: JWT auth with refresh token rotation"
PR merged (squash). Branch deleted.

Metrics committed. Worktree cleaned.
✅ Feature complete.
```

## 适用场景

在 `/plan-pipeline:validate` 确认所有问题已解决后使用。永远不要跳过验证——执行未经验证的计划会跳过独立审查，而该审查平均能发现约 18 个问题。

## 流水线位置

```
/plan-pipeline:ceo-review    → 产品方向锁定
/plan-pipeline:eng-review    → 架构锁定
/plan-pipeline:start         → 生成实现计划
/plan-pipeline:validate      → 执行前验证
/plan-pipeline:execute       → 执行至 PR 合并          ← 当前位置
```
