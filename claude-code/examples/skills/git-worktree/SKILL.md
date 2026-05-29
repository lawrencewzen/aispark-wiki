> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: git-worktree
description: 创建隔离的 Git 工作树，无需切换分支即可进行功能开发
argument-hint: "<branch_name> [--from <base>]"
effort: medium
disable-model-invocation: true
---

# Git 工作树设置

创建隔离的 Git 工作树，无需切换分支即可进行功能开发。

**核心原则：** 智能目录选择 + 符号链接优化 + 后台验证 = 快速、可靠的隔离环境。

**依赖要求：** Git 2.5.0+（2015 年 7 月）

**配套命令：** [`/git-worktree-status`](./git-worktree-status.md) | [`/git-worktree-remove`](./git-worktree-remove.md) | [`/git-worktree-clean`](./git-worktree-clean.md)

## 流程

1. **验证分支名称**：检查命名规范与冲突
2. **检查现有目录**：`.worktrees/` 或 `worktrees/`
3. **验证 .gitignore**：确保工作树目录已被忽略
4. **创建工作树**：`git worktree add`
5. **符号链接依赖**：复用主工作树的 `node_modules/`
6. **检测数据库提供商**：检查是否支持数据库分支
7. **安装依赖**：自动检测包管理器（未使用符号链接时）
8. **运行后台验证**：在后台进行类型检查 + 测试
9. **报告位置**：确认就绪状态

## 标志

| 标志 | 效果 |
|------|--------|
| `--fast` | 跳过依赖安装和基准测试 |
| `--isolated` | 全新安装 `node_modules`（不使用符号链接） |
| `--skip-install` | 跳过依赖安装，保留基准测试 |

## 分支名称验证

```bash
# Auto-prefix based on naming convention
# "auth" → "feat/auth" (default prefix)
# "fix/login-bug" → kept as-is
# "refactor/db-layer" → kept as-is

# Accepted prefixes: feat/, fix/, refactor/, chore/, docs/, test/, perf/
# If no prefix → default to feat/

# Reject invalid characters
echo "$BRANCH_NAME" | grep -qE '^[a-zA-Z0-9/_-]+$' || exit 1

# Check branch doesn't already exist
git show-ref --verify --quiet "refs/heads/$BRANCH_NAME" && echo "Branch already exists" && exit 1
```

## 目录选择

### 优先级顺序

```bash
# 1. Check existing directories
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative

# 2. Check CLAUDE.md for preference
grep -i "worktree.*director" CLAUDE.md 2>/dev/null

# 3. Ask user if neither exists
```

**两者同时存在时：** `.worktrees/` 优先。

## 安全验证

**对于项目本地目录：**

```bash
# Check if directory in .gitignore
grep -q "^\.worktrees/$" .gitignore || grep -q "^worktrees/$" .gitignore
```

**若不在 .gitignore 中：**
1. 向 .gitignore 添加该行
2. 提交变更
3. 继续创建工作树

**为何关键：** 防止意外提交工作树内容。

## 创建步骤

```bash
# 1. Detect project name
project=$(basename "$(git rev-parse --show-toplevel)")

# 2. Create worktree with new branch
git worktree add .worktrees/$BRANCH_NAME -b $BRANCH_NAME

# 3. Navigate
cd .worktrees/$BRANCH_NAME
```

## 依赖优化（Node.js）

**默认行为：** 从主工作树符号链接 `node_modules`，避免重复安装（节省约 30 秒）。

```bash
# Symlink node_modules (default, unless --isolated)
if [ -d "../../node_modules" ] && [ ! "$ISOLATED" = true ]; then
  ln -s "$(cd ../.. && pwd)/node_modules" node_modules
  echo "Symlinked node_modules from main worktree"
fi

# With --isolated: fresh install
if [ "$ISOLATED" = true ]; then
  pnpm install   # or npm/yarn based on lockfile detection
fi
```

**何时使用 `--isolated`：**
- 需要不同包版本的 Schema 变更
- 测试依赖升级
- 调试 `node_modules` 问题

## 自动检测设置（多技术栈）

```bash
# Node.js (if not symlinked)
if [ -f package.json ] && [ ! -L node_modules ]; then
  pnpm install   # Detect from lockfile: pnpm-lock.yaml / yarn.lock / package-lock.json
fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 后台验证

**无需阻塞等待完整测试套件，在后台运行验证：**

```bash
# Create log directory
mkdir -p .worktree-logs

# Background type check (Node.js)
if [ -f tsconfig.json ]; then
  npx tsc --noEmit > .worktree-logs/typecheck.log 2>&1 &
  echo "Type check running in background (check with /git-worktree-status)"
fi

# Background test run
if [ -f package.json ]; then
  npx vitest run --reporter=json > .worktree-logs/tests.log 2>&1 &
  echo "Tests running in background (check with /git-worktree-status)"
fi
```

**使用 `--fast`：** 跳过所有验证。

## 最终报告

```
Worktree ready at <full-path>
Branch: feat/auth (created from main)
Dependencies: symlinked from main worktree
Background checks: type check + tests running
Check status: /git-worktree-status

Ready to implement <feature-name>
```

## 数据库分支建议

**工作树创建完成后，检测数据库提供商并建议隔离方案。**

### 快速命令参考

| 提供商 | 建议命令 |
|----------|-------------------|
| **Neon** | `neonctl branches create --name <branch> --parent main` |
| **PlanetScale** | `pscale branch create <db> <branch>` |
| **本地 Postgres** | `psql -c "CREATE SCHEMA <schema>;"` |
| **其他** | 手动设置或共享数据库 |

**示例输出：**

```
Worktree created at .worktrees/feat/auth

DB Isolation: neonctl branches create --name feat-auth --parent main
   Then update .env with new DATABASE_URL
   Full guide: ../workflows/database-branch-setup.md
```

### .worktreeinclude 设置

**对于环境变量至关重要：**

```bash
# .worktreeinclude (at project root)
.env
.env.local
.env.development
**/.claude/settings.local.json
```

**原因：** 若不设置，`.env` 文件将不会复制到工作树。

### 何时创建数据库分支

| 场景 | 是否创建分支 |
|----------|---------------|
| Schema 迁移 | 是 |
| 数据模型重构 | 是 |
| Bug 修复（无 Schema 变更） | 否 |
| 性能实验 | 是 |

**参阅：** [数据库分支设置指南](../workflows/database-branch-setup.md) 了解完整工作流。

## 快速参考

| 情况 | 操作 |
|-----------|--------|
| `.worktrees/` 存在 | 使用它（验证 .gitignore） |
| `worktrees/` 存在 | 使用它（验证 .gitignore） |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 检查 CLAUDE.md，然后询问用户 |
| 不在 .gitignore 中 | 立即添加并提交 |
| 无分支前缀 | 自动添加 `feat/` 前缀 |
| Node.js 项目 | 默认符号链接 `node_modules` |
| `--fast` 标志 | 跳过安装 + 测试 |
| `--isolated` 标志 | 全新安装 `node_modules` |
| 检测到 Neon | 建议 `neonctl branches create` |
| 检测到 PlanetScale | 建议 `pscale branch create` |
| 无 .worktreeinclude | 使用 `.env` 模式创建 |

## 常见错误

**跳过 .gitignore 验证**
- 工作树内容被跟踪，污染 git 状态

**假定目录位置**
- 遵循优先级：已有目录 > CLAUDE.md > 询问用户

**在每个工作树中都完整安装 node_modules**
- 浪费磁盘空间和时间。默认使用符号链接，仅在必要时使用 `--isolated`

**未将 .env 复制到工作树**
- 症状：Claude 报错 "DATABASE_URL not found"
- 修复：将 `.env` 添加到 `.worktreeinclude`

**在 Schema 变更时使用共享数据库**
- 症状：迁移冲突，开发环境损坏
- 修复：在修改 Schema 前创建数据库分支

## 使用方式

```
/git-worktree auth
/git-worktree fix/session-bug
/git-worktree feature/new-api --fast
/git-worktree refactor/db-layer --isolated
```

Branch name: $ARGUMENTS