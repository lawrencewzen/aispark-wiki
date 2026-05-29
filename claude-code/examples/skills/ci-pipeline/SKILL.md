> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ci-pipeline
description: 推送当前分支并返回流水线追踪 URL（GitLab 或 GitHub Actions）
argument-hint: "[--force | --draft]"
allowed-tools: [Bash]
model: haiku
effort: low
disable-model-invocation: true
---

# /ci:pipeline — 推送并触发流水线

推送当前分支并返回流水线追踪链接。

## 执行流程

```bash
BRANCH=$(git branch --show-current)

# 1. 安全检查
if echo "$BRANCH" | grep -qE "^(main|master|production)$"; then
  echo "❌ Direct push to $BRANCH is not allowed. Create a feature/fix branch."
  exit 1
fi

# 2. 检查未提交的变更
UNCOMMITTED=$(git status --porcelain | wc -l | tr -d ' ')
if [ "$UNCOMMITTED" -gt 0 ]; then
  echo "⚠️  $UNCOMMITTED uncommitted file(s):"
  git status --short
  echo ""
  echo "Commit first with /commit or git add + git commit"
  exit 0
fi

# 3. 推送
echo "Pushing → origin/$BRANCH"
git push origin "$BRANCH" 2>&1
```

## 流水线 URL

### GitLab CI

```bash
REMOTE=$(git remote get-url origin 2>/dev/null || echo "")
WEB_URL=$(echo "$REMOTE" | sed 's/git@gitlab\.com:/https:\/\/gitlab.com\//' | sed 's/\.git$//')
echo ""
echo "✅ Push OK"
echo "   Pipeline: $WEB_URL/-/pipelines?ref=$BRANCH"

# 通过 glab 查看实时状态（如已安装）
if command -v glab &>/dev/null; then
  sleep 3
  glab ci status --branch "$BRANCH" 2>/dev/null || true
fi
```

### GitHub Actions

```bash
REMOTE=$(git remote get-url origin 2>/dev/null || echo "")
WEB_URL=$(echo "$REMOTE" | sed 's/git@github\.com:/https:\/\/github.com\//' | sed 's/\.git$//')
echo ""
echo "✅ Push OK"
echo "   Actions: $WEB_URL/actions?query=branch%3A$BRANCH"

# 通过 gh 查看实时状态（如已安装）
if command -v gh &>/dev/null; then
  sleep 5
  gh run list --branch "$BRANCH" --limit 3 2>/dev/null || true
fi
```

## 预期输出

```
Pushing → origin/feat/add-payment-retry

Enumerating objects: 12, done.
...

✅ Push OK
   Pipeline: https://gitlab.com/org/my-app/-/pipelines?ref=feat/add-payment-retry

Pipeline status (after 3s):
  ⏳ lint       running
  ⏸️  test       waiting
  ⏸️  deploy     waiting
```

## 使用方式

```
/ci:pipeline
```

目标：$ARGUMENTS
