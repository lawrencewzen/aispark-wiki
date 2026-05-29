> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: plan-pipeline
description: "编排完整规划流水线：产品方向（ceo-review）→ 架构设计（eng-review）→ 实现计划（start）→ 验证（validate）→ 执行（execute）。可单独运行各阶段，也可让编排者协调完整流程。"
allowed-tools: "Read, Write, Bash, Task"
effort: high
---

# 规划流水线编排者

编排从规划到执行的完整流水线。可以运行完整流水线，也可以单独运行某个孤立阶段。

## 阶段说明

| 阶段 | Skill | 用途 |
|------|-------|------|
| 1 | `/plan-pipeline:ceo-review` | 挑战需求简报，锁定产品方向 |
| 2 | `/plan-pipeline:eng-review` | 锁定架构、图表和测试矩阵 |
| 3 | `/plan-pipeline:start` | 5 阶段规划：PRD、调研、ADR、任务清单 |
| 4 | `/plan-pipeline:validate` | 编写代码前的双层验证 |
| 5 | `/plan-pipeline:execute` | Worktree 隔离、并行智能体、质量门控、PR |

## 使用方式

```
/plan-pipeline                     # 完整流水线，会询问上下文
/plan-pipeline --from=start        # 跳过门控，从规划阶段开始
/plan-pipeline --from=validate     # 验证已有计划
/plan-pipeline --from=execute      # 执行已验证的计划
```

## 各阶段适用时机

**ceo-review** — 在方向未确定前用于任何重要功能。当需求非常具体时尤其有价值（具体性意味着解决方案空间已收窄）。

**eng-review** — 方向锁定后使用。对于含异步组件、外部依赖或多步骤流程的功能为必要步骤。

**start** — 用于任何涉及超过 2 个文件或架构决策的非平凡功能。

**validate** — 执行前必须运行。验证的成本相比在执行中途发现问题的代价微不足道。

**execute** — 在 validate 确认所有问题已解决后运行。

## 工作流

1. **收集上下文** — 我们要构建什么，从哪个阶段开始？
2. **ceo-review** — 产品方向门控（可通过 `--from=eng-review` 或更后的阶段跳过）
3. **eng-review** — 架构门控（可通过 `--from=start` 或更后的阶段跳过）
4. **检查点** — 在规划前请用户确认方向和架构
5. **start** — 运行 5 阶段规划，产出 `docs/plans/plan-{name}.md`
6. **检查点** — 在验证前展示计划供审阅
7. **validate** — 双层验证（结构验证 + 专项智能体）
8. **execute** — Worktree 隔离 → 并行智能体 → 质量门控 → PR

## 依赖关系图

```
   ceo-review
        |
   eng-review
        |
      start
        |
    validate
        |
    execute
```

## 说明

每个阶段将其输出写入磁盘后，下一阶段才开始。如果流水线中断，使用正确的阶段名通过 `--from=<stage>` 恢复。所有决策记录在 `docs/plans/plan-{name}.md` 及对应的 `docs/adr/` 目录中。
