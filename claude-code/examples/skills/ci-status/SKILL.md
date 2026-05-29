> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ci-status
description: 显示当前分支的流水线状态——GitLab CI 或 GitHub Actions
argument-hint: "[pr_number optional]"
allowed-tools: [Bash]
model: haiku
effort: low
disable-model-invocation: true
---

# /ci:status — 流水线状态

快速查看当前分支的 CI 流水线快照。

## 流程

```bash
# 1. 当前分支
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"

# 2. 最近提交
git log -3 --oneline
```

## GitLab CI

```bash
# With glab CLI (recommended)
if command -v glab &>/dev/null; then
  glab ci status --branch "$BRANCH"
else
  # Fallback: show URL
  REMOTE=$(git remote get-url origin 2>/dev/null || echo "")
  if [ -n "$REMOTE" ]; then
    WEB_URL=$(echo "$REMOTE" | sed 's/git@gitlab\.com:/https:\/\/gitlab.com\//' | sed 's/\.git$//')
    echo "Pipeline: $WEB_URL/-/pipelines?ref=$BRANCH"
  fi
fi

# If MR number provided
if [ -n "$ARGUMENTS" ] && command -v glab &>/dev/null; then
  glab mr view "$ARGUMENTS"
fi
```

## GitHub Actions

```bash
# With gh CLI (recommended)
if command -v gh &>/dev/null; then
  gh run list --branch "$BRANCH" --limit 5
else
  # Fallback: show URL
  REMOTE=$(git remote get-url origin 2>/dev/null || echo "")
  if [ -n "$REMOTE" ]; then
    WEB_URL=$(echo "$REMOTE" | sed 's/git@github\.com:/https:\/\/github.com\//' | sed 's/\.git$//')
    echo "Actions: $WEB_URL/actions?query=branch%3A$BRANCH"
  fi
fi

# If PR number provided
if [ -n "$ARGUMENTS" ] && command -v gh &>/dev/null; then
  gh pr checks "$ARGUMENTS"
fi
```

## 预期输出

```
Branch: feat/add-payment-retry

Last commits:
  a1b2c3d feat(payments): add retry logic with exponential backoff
  e4f5g6h test(payments): add Vitest coverage on retry scenarios
  i7j8k9l chore: update pnpm lockfile

Pipeline: running
  ✅ lint          30s
  ✅ typecheck     45s
  ⏳ test          running...
  ⏸️  deploy        waiting

URL: https://gitlab.com/org/my-app/-/pipelines?ref=feat/add-payment-retry
```

## 用法

```
/ci:status
/ci:status 42    # MR 或 PR 编号 42
```

目标：$ARGUMENTS
