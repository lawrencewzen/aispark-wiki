> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: git-worktree-remove
description: 安全移除 git worktree，包含分支清理和安全检查
argument-hint: <worktree_name>
effort: low
disable-model-invocation: true
---

# Git Worktree 移除

安全移除单个 git worktree，包含分支清理、合并验证和数据库分支清理。

**核心原则：** 先执行安全检查，再干净地移除 worktree + 分支 + 数据库资源。

**所属套件：** [Worktree 生命周期套件](./git-worktree.md) | [`/git-worktree`](./git-worktree.md) | [`/git-worktree-status`](./git-worktree-status.md) | [`/git-worktree-clean`](./git-worktree-clean.md)

## 流程

1. **验证目标**：确认要移除的 worktree
2. **安全检查**：保护 main/develop 分支
3. **检查合并状态**：若分支有未合并变更则发出警告
4. **检查未提交变更**：若 worktree 有脏状态则发出警告
5. **移除 Worktree**：`git worktree remove`
6. **删除本地分支**：`git branch -d`（或确认后使用 `-D`）
7. **删除远程分支**：`git push origin --delete`（需确认）
8. **数据库清理提示**：如适用，提示删除数据库分支
9. **清理悬空引用**：`git worktree prune`

## 安全检查

### 受保护分支

```bash
# 以下分支的 worktree 永远不会被移除（可配置）
PROTECTED_BRANCHES="main master develop staging production"

if echo "$PROTECTED_BRANCHES" | grep -qw "$BRANCH"; then
  echo "BLOCKED: Cannot remove worktree for protected branch '$BRANCH'"
  echo "Protected branches: $PROTECTED_BRANCHES"
  exit 1
fi
```

### 未提交变更

```bash
cd "$WORKTREE_PATH"
if [ -n "$(git status --porcelain)" ]; then
  echo "WARNING: Worktree has uncommitted changes:"
  git status --short
  echo ""
  echo "Options:"
  echo "  1. Commit changes first"
  echo "  2. Force remove (--force)"
  echo "  3. Cancel"
  # Wait for user decision
fi
```

### 合并状态

```bash
# 检查分支是否已合并到 main
MAIN_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@')

if git merge-base --is-ancestor "$BRANCH" "$MAIN_BRANCH" 2>/dev/null; then
  echo "Branch '$BRANCH' is merged into $MAIN_BRANCH. Safe to delete."
  MERGED=true
else
  echo "WARNING: Branch '$BRANCH' is NOT merged into $MAIN_BRANCH."
  echo "You may lose work if you delete this branch."
  MERGED=false
fi
```

## 移除步骤

```bash
# 1. 移除 worktree
git worktree remove "$WORKTREE_PATH"
# 若有脏状态且用户确认强制移除：
# git worktree remove --force "$WORKTREE_PATH"

# 2. 删除本地分支
if [ "$MERGED" = true ]; then
  git branch -d "$BRANCH"
else
  echo "Delete unmerged branch '$BRANCH'? (requires confirmation)"
  # 确认后：
  git branch -D "$BRANCH"
fi

# 3. 删除远程分支（需确认）
if git ls-remote --heads origin "$BRANCH" | grep -q "$BRANCH"; then
  echo "Delete remote branch 'origin/$BRANCH'?"
  # 确认后：
  git push origin --delete "$BRANCH"
fi

# 4. 清理悬空引用
git worktree prune
```

## 数据库分支清理

**移除 worktree 后，提示清理关联的数据库分支：**

```bash
# 检测数据库提供商（与 /git-worktree 逻辑相同）
if [ -f ".env" ] && grep -q "neon" ".env"; then
  echo ""
  echo "DB Cleanup: neonctl branches delete $BRANCH_SLUG"
elif [ -f ".pscale.yml" ]; then
  echo ""
  DB_NAME=$(grep 'database:' .pscale.yml | awk '{print $2}')
  echo "DB Cleanup: pscale branch delete $DB_NAME $BRANCH_SLUG"
elif [ -f ".env" ] && grep -q "postgresql" ".env"; then
  echo ""
  echo "DB Cleanup: psql \$DATABASE_URL -c \"DROP SCHEMA ${BRANCH_SLUG} CASCADE;\""
fi
```

## 报告格式

**成功移除（已合并分支）：**

```
Removed worktree: .worktrees/feat/auth
  Worktree directory: deleted
  Local branch feat/auth: deleted (was merged)
  Remote branch origin/feat/auth: deleted
  References: pruned

DB reminder: neonctl branches delete feat-auth
```

**带警告的移除（未合并分支）：**

```
Removed worktree: .worktrees/feat/experimental
  Worktree directory: deleted
  Local branch feat/experimental: deleted (was NOT merged - forced)
  Remote branch: no remote branch found
  References: pruned

WARNING: Branch was not merged. Changes may be lost.
Last commit: a1b2c3d "WIP: experimental auth flow"
```

## 标志

| 标志 | 效果 |
|------|--------|
| `--force` | 跳过未提交变更警告 |
| `--keep-branch` | 移除 worktree 但保留分支 |
| `--keep-remote` | 不删除远程分支 |

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| 分支已合并 | 安全删除（branch -d） |
| 分支未合并 | 警告 + 需要确认（branch -D） |
| 有未提交变更 | 警告 + 提供强制/取消选项 |
| 受保护分支（main/develop） | 阻止移除 |
| 存在远程分支 | 询问是否删除远程分支 |
| 检测到数据库分支 | 提示精确命令 |
| 悬空引用 | 自动清理 |

## 常见错误

**移除 main/develop 的 worktree**
- 始终被安全检查阻止。如需要，可重新配置受保护分支。

**未检查即删除未合并分支**
- 始终验证合并状态。未合并的分支需要显式使用 `--force` 或 `-D`。

**忘记清理数据库分支**
- 会留下占用资源的孤立数据库分支。命令会自动提示。

**用 `rm -rf` 代替 `git worktree remove`**
- 会在 `.git/worktrees/` 中留下悬空的 worktree 引用。始终使用 git 命令。

## 使用方式

```
/git-worktree-remove feat/auth
/git-worktree-remove fix/login-bug --force
/git-worktree-remove refactor/db --keep-branch
```

分支或 worktree 路径：$ARGUMENTS
