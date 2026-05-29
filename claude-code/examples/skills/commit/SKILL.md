> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: commit
description: 为已暂存的改动生成符合约定式提交规范的提交信息
argument-hint: "[--amend] [message]"
effort: low
disable-model-invocation: true
---

# 约定式提交

为已暂存的改动生成符合规范的提交信息。

## 操作步骤

1. 执行 `git diff --cached` 查看已暂存的改动
2. 分析改动性质
3. 按以下格式生成提交信息

## 提交信息格式

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### 类型说明
- `feat`：新功能
- `fix`：Bug 修复
- `docs`：仅文档变更
- `style`：格式调整，缺少分号等（不影响功能）
- `refactor`：既不修复 bug 也不新增功能的代码重构
- `perf`：性能优化
- `test`：补充缺失的测试
- `chore`：维护性任务

### 规则
- 主题行：祈使语气，不加句号，最多 50 个字符
- 正文：说明改了什么（WHAT）以及为什么改（WHY），而不是怎么改的（HOW）
- 页脚：破坏性变更、关联 issue

## 示例

```
feat(auth): add password reset functionality

Implement password reset flow with email verification.
Users can now request a reset link and set new password.

Closes #123
```

```
fix(api): prevent race condition in order processing

Add mutex lock to ensure orders are processed sequentially.
This fixes duplicate charge issues reported by users.

Fixes #456
```

```
refactor(cart): extract pricing logic to separate module

No functional changes. Improves testability and
separates concerns for future discount feature.
```

## 执行

分析已暂存的改动后，给出提交信息建议。
在执行 `git commit -m "..."` 前请求用户确认。

$ARGUMENTS
