> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: validate-changes
description: 在提交前使用 LLM 作为评判者对暂存的变更进行评估
effort: low
disable-model-invocation: true
---

# 提交前验证变更

使用 output-evaluator 智能体对已暂存的 git 变更进行评估，在提交之前发现问题。

## 流程

### 第一步：检查是否有暂存变更

运行 `git diff --cached --stat` 查看已暂存的内容。如果没有任何暂存，告知用户并退出。

### 第二步：获取完整的 diff

运行 `git diff --cached` 获取所有暂存变更的完整 diff。

### 第三步：调用评估器

使用 Task 工具启动 `output-evaluator` 智能体，并传入 diff：

```
Evaluate these staged changes for correctness, completeness, and safety.
Return a JSON verdict with scores and issues.

Changes:
[paste the git diff here]
```

### 第四步：解析结果并采取行动

根据评估结果：

**如果结果为 APPROVE（通过）：**
- 告知用户变更已通过评估
- 展示摘要和评分
- 询问是否继续提交

**如果结果为 NEEDS_REVIEW（需复查）：**
- 展示发现的所有问题（按严重程度分组）
- 展示评估器的修改建议
- 询问用户如何处理：
  - 修复问题后重新评估
  - 直接提交（确认接受风险）
  - 放弃提交

**如果结果为 REJECT（拒绝）：**
- 明确告知变更被拒绝
- 展示导致拒绝的关键问题
- 不要提供直接提交的选项
- 给出具体的修复建议

### 第五步：提交（如已通过）

若用户确认，按标准提交流程创建提交。

## 使用示例

```
/validate-changes
```

输出：
```
Evaluating 3 staged files...

VERDICT: NEEDS_REVIEW

Scores:
  Correctness:  8/10
  Completeness: 6/10
  Safety:       9/10

Issues Found:
  [MEDIUM] src/api/handler.ts:45
    Missing error handling for network failures

  [LOW] src/utils/format.ts:12
    Consider adding input validation

Suggestion: Add try-catch around the fetch call in handler.ts

How would you like to proceed?
  1. Fix issues and re-evaluate
  2. Commit anyway (1 medium issue)
  3. Abort
```

## 费用说明

此命令会调用 LLM 评估，会消耗 API Token：
- **典型费用**：每次评估 $0.01-0.05（使用 Haiku 模型）
- **较大的 diff**：由于 Token 用量增加，费用可能更高

## 何时使用

- 重大代码变更提交之前
- 在不熟悉的代码库区域工作时
- 变更涉及安全敏感代码时
- 推送到共享分支之前

## 何时跳过

- 琐碎变更（改错别字、格式调整）
- 仅修改文档
- 已经手动仔细审查过的情况
- 在功能分支上快速迭代时

## 与 Git 钩子集成

如需在每次提交时自动评估，请参阅 `pre-commit-evaluator.sh` 钩子。
本命令是手动替代方案，适用于需要自主控制评估时机的场景。
