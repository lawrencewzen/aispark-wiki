> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: git-worktree-clean
description: 清理过期 Git 工作树，附带已合并分支检测和磁盘占用报告
argument-hint: "[--dry-run]"
effort: low
disable-model-invocation: true
---

# Git 工作树清理

批量清理过期的 Git 工作树。安全移除已合并分支，报告磁盘使用情况，并对未合并分支进行交互式处理。

**核心原则：** 自动清理已合并的工作树，对未合并的工作树进行交互式审查，并始终报告已回收的空间。

**所属系列：** [工作树生命周期套件](./git-worktree.md) | [`/git-worktree`](./git-worktree.md) | [`/git-worktree-status`](./git-worktree-status.md) | [`/git-worktree-remove`](./git-worktree-remove.md)

## 处理流程

1. **列出所有工作树**：`git worktree list`
2. **逐一分类**：已合并 / 未合并 / 受保护
3. **计算磁盘占用**：每个工作树的大小
4. **自动模式**：移除所有已合并的工作树（安全）
5. **交互模式**：逐一审查未合并的工作树
6. **数据库清理提醒**：列出待清理的数据库分支
7. **报告**：已执行操作及回收空间的汇总

## 参数标志

| 标志 | 效果 |
|------|--------|
| `--dry-run` | 预览待清理内容，不做任何变更 |
| `--all` | 包含未合并的工作树（每个需逐一确认） |
| `--force` | 不需确认直接移除所有工作树（危险） |

## 工作树发现

```bash
# 获取主分支名称
MAIN_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
MAIN_BRANCH=${MAIN_BRANCH:-main}

# 受保护分支（永不自动清理）
PROTECTED="main master develop staging production"

# 列出所有工作树（跳过主工作树）
git worktree list --porcelain | while read line; do
  # 解析工作树路径和分支
  # 跳过主工作树（第一条记录）
done
```

## 分类逻辑

```bash
for WORKTREE in $WORKTREES; do
  BRANCH=$(git -C "$WORKTREE" rev-parse --abbrev-ref HEAD)

  # 跳过受保护分支
  if echo "$PROTECTED" | grep -qw "$BRANCH"; then
    echo "PROTECTED: $BRANCH (skipped)"
    continue
  fi

  # 检查合并状态
  if git merge-base --is-ancestor "$BRANCH" "$MAIN_BRANCH" 2>/dev/null; then
    echo "MERGED: $BRANCH → safe to remove"
    MERGED_LIST="$MERGED_LIST $WORKTREE"
  else
    echo "UNMERGED: $BRANCH → requires review"
    UNMERGED_LIST="$UNMERGED_LIST $WORKTREE"
  fi
done
```

## 磁盘占用计算

```bash
for WORKTREE in $ALL_WORKTREES; do
  # 计算大小，排除软链接的 node_modules
  SIZE=$(du -sh --exclude='node_modules' "$WORKTREE" 2>/dev/null | cut -f1)
  # 或在 macOS 上：
  SIZE=$(du -sh -I 'node_modules' "$WORKTREE" 2>/dev/null | cut -f1)
  echo "  $WORKTREE: $SIZE"
done
```

## 试运行模式

```bash
# --dry-run：预览将发生的操作，不做任何变更

echo "=== Dry Run ==="
echo ""
echo "Would remove (merged):"
for WT in $MERGED_LIST; do
  echo "  $WT ($BRANCH) - $SIZE"
done
echo ""
echo "Would ask about (unmerged):"
for WT in $UNMERGED_LIST; do
  echo "  $WT ($BRANCH) - $SIZE - last commit: $(git log -1 --format='%s' $BRANCH)"
done
echo ""
echo "Total space to reclaim: $TOTAL_SIZE"
echo ""
echo "Run without --dry-run to execute."
```

## 自动模式（默认）

**仅移除已合并的工作树，默认安全。**

```bash
echo "Cleaning merged worktrees..."

for WORKTREE in $MERGED_LIST; do
  BRANCH=$(git -C "$WORKTREE" rev-parse --abbrev-ref HEAD)

  # 移除工作树
  git worktree remove "$WORKTREE"

  # 删除本地分支
  git branch -d "$BRANCH" 2>/dev/null

  # 删除远程分支
  git push origin --delete "$BRANCH" 2>/dev/null

  echo "  Removed: $WORKTREE ($BRANCH)"
done

# 报告未合并的工作树（不处理）
if [ -n "$UNMERGED_LIST" ]; then
  echo ""
  echo "Unmerged worktrees (kept):"
  for WT in $UNMERGED_LIST; do
    echo "  $WT - use /git-worktree-remove or --all to review"
  done
fi
```

## 交互模式（--all）

**逐一审查未合并的工作树：**

```bash
for WORKTREE in $UNMERGED_LIST; do
  BRANCH=$(git -C "$WORKTREE" rev-parse --abbrev-ref HEAD)
  LAST_COMMIT=$(git log -1 --format='%h %s (%cr)' "$BRANCH")
  AHEAD=$(git rev-list --count "$MAIN_BRANCH".."$BRANCH")

  echo ""
  echo "Unmerged: $WORKTREE"
  echo "  Branch: $BRANCH ($AHEAD commits ahead of $MAIN_BRANCH)"
  echo "  Last commit: $LAST_COMMIT"
  echo "  Size: $SIZE"
  echo ""
  echo "  [r]emove  [k]eep  [s]kip remaining"

  # 等待用户对每个工作树做出决定
done
```

## 报告格式

**清理完成后：**

```
=== Worktree Cleanup Report ===

Removed (merged):
  .worktrees/feat/auth (feat/auth) - 2.3 MB
  .worktrees/fix/login-bug (fix/login-bug) - 1.1 MB
  .worktrees/chore/deps-update (chore/deps-update) - 0.8 MB

Kept (unmerged):
  .worktrees/feat/experimental (feat/experimental) - 4.2 MB
    Last commit: a1b2c3d "WIP: new auth flow" (3 days ago)

Kept (protected):
  .worktrees/develop (develop)

Space reclaimed: 4.2 MB
Worktrees remaining: 2
References pruned: yes

DB branches to clean:
  neonctl branches delete feat-auth
  neonctl branches delete fix-login-bug
  neonctl branches delete chore-deps-update
```

**试运行报告：**

```
=== Dry Run - No Changes Made ===

Would remove (3 merged):
  .worktrees/feat/auth - 2.3 MB
  .worktrees/fix/login-bug - 1.1 MB
  .worktrees/chore/deps-update - 0.8 MB

Would keep (1 unmerged):
  .worktrees/feat/experimental - 4.2 MB

Would keep (1 protected):
  .worktrees/develop

Potential space savings: 4.2 MB
```

## 快速参考

| 场景 | 操作 |
|-----------|--------|
| 默认（无标志） | 仅移除已合并的工作树 |
| `--dry-run` | 预览，不做变更 |
| `--all` | 已合并（自动）+ 未合并（交互式） |
| `--force` | 移除除受保护外的所有内容 |
| 受保护分支 | 始终保留 |
| 已合并分支 | 自动移除 |
| 未合并分支 | 保留（默认）或交互式处理（--all） |
| 检测到数据库分支 | 带精确命令的清理提醒 |

## 常见误操作

**在 `--dry-run` 预览前直接使用 `--force`**
- 强制清理前务必先用 `--dry-run` 预览

**忘记清理数据库分支**
- 工作树清理不会自动删除数据库分支，请按提醒命令操作。

**不定期清理**
- 过期工作树会持续占用磁盘空间，建议每周运行 `/git-worktree-clean --dry-run`。

## 使用方式

```
/git-worktree-clean
/git-worktree-clean --dry-run
/git-worktree-clean --all
```

标志参数：$ARGUMENTS
