> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: git-worktree-status
description: 检查在 git worktree 中运行的后台验证任务状态
effort: low
disable-model-invocation: true
---

# Git Worktree 状态

检查由 `/git-worktree` 启动的后台验证任务（类型检查、测试、构建）。

**核心原则：** 在不中断开发流程的前提下，对 worktree 健康状态提供非阻塞式反馈。

**所属：** [Worktree 生命周期套件](./git-worktree.md) | [`/git-worktree`](./git-worktree.md) | [`/git-worktree-remove`](./git-worktree-remove.md) | [`/git-worktree-clean`](./git-worktree-clean.md)

## 流程

1. **检测当前 Worktree**：验证当前是否在 git worktree 内
2. **检查日志文件**：读取 `.worktree-logs/` 中的后台任务结果
3. **解析结果**：提取通过/失败数量、错误信息
4. **报告状态**：带颜色的摘要，附可操作的后续步骤

## Worktree 检测

```bash
# 检查是否在 worktree 内（而非主仓库）
git rev-parse --git-common-dir 2>/dev/null | grep -q "\.git/worktrees" || {
  echo "Not inside a worktree. Use from a worktree directory."
  exit 1
}

# 获取 worktree 信息
WORKTREE_PATH=$(git rev-parse --show-toplevel)
BRANCH=$(git rev-parse --abbrev-ref HEAD)
MAIN_REPO=$(git rev-parse --git-common-dir | sed 's|/\.git/worktrees/.*||')
```

## 后台任务检查

### 类型检查状态

```bash
LOG=".worktree-logs/typecheck.log"

if [ -f "$LOG" ]; then
  if grep -q "error TS" "$LOG"; then
    ERROR_COUNT=$(grep -c "error TS" "$LOG")
    echo "Type check: FAIL ($ERROR_COUNT errors)"
    # Show first 5 errors
    grep "error TS" "$LOG" | head -5
  else
    echo "Type check: PASS"
  fi
elif pgrep -f "tsc --noEmit" > /dev/null; then
  echo "Type check: RUNNING..."
else
  echo "Type check: NOT RUN"
fi
```

### 测试状态

```bash
LOG=".worktree-logs/tests.log"

if [ -f "$LOG" ]; then
  if grep -q '"numFailedTests":0' "$LOG"; then
    TOTAL=$(grep -o '"numTotalTests":[0-9]*' "$LOG" | cut -d: -f2)
    echo "Tests: PASS ($TOTAL tests)"
  else
    FAILED=$(grep -o '"numFailedTests":[0-9]*' "$LOG" | cut -d: -f2)
    echo "Tests: FAIL ($FAILED failures)"
    # Show failed test names
    grep '"fullName"' "$LOG" | head -5
  fi
elif pgrep -f "vitest run" > /dev/null; then
  echo "Tests: RUNNING..."
else
  echo "Tests: NOT RUN"
fi
```

### 构建状态

```bash
LOG=".worktree-logs/build.log"

if [ -f "$LOG" ]; then
  if [ $? -eq 0 ]; then
    echo "Build: PASS"
  else
    echo "Build: FAIL"
    tail -10 "$LOG"
  fi
elif pgrep -f "cargo build\|next build\|go build" > /dev/null; then
  echo "Build: RUNNING..."
else
  echo "Build: NOT RUN"
fi
```

## 报告格式

```
Worktree Status: .worktrees/feat/auth
Branch: feat/auth (from main, 3 commits ahead)

Checks:
  Type check:  PASS
  Tests:       PASS (142 tests)
  Build:       NOT RUN

Dependencies: symlinked from main
Disk usage: 2.3 MB (excl. node_modules)

Log files: .worktree-logs/
```

**检测到失败时：**

```
Worktree Status: .worktrees/feat/auth
Branch: feat/auth (from main, 3 commits ahead)

Checks:
  Type check:  FAIL (3 errors)
    src/auth.ts:42 - error TS2345: Argument of type 'string' is not assignable
    src/auth.ts:67 - error TS2304: Cannot find name 'AuthConfig'
    src/middleware.ts:12 - error TS7006: Parameter 'req' implicitly has an 'any' type
  Tests:       FAIL (2 failures)
    auth.test.ts > should validate token
    auth.test.ts > should reject expired token
  Build:       NOT RUN

Action: Fix type errors before proceeding. Run `npx tsc --noEmit` for full output.
```

## 日志管理

```bash
# 清理旧日志（适用于重新运行检查）
rm -rf .worktree-logs/*.log

# 重新运行所有检查
npx tsc --noEmit > .worktree-logs/typecheck.log 2>&1 &
npx vitest run --reporter=json > .worktree-logs/tests.log 2>&1 &
```

## 快速参考

| 情况 | 输出 |
|------|------|
| 所有检查通过 | 绿色状态，可以继续工作 |
| 检查仍在运行 | "RUNNING..." 及 PID |
| 发现类型错误 | 错误数量 + 前5个错误 |
| 测试失败 | 失败数量 + 失败测试名称 |
| 未找到日志 | "NOT RUN"（使用了 `--fast` 或日志已删除） |
| 不在 worktree 内 | 附操作说明的错误消息 |

## 用法

```
/git-worktree-status
```

无需任何参数。在任意 worktree 目录内运行即可。
