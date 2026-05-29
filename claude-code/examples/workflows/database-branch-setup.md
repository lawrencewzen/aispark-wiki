> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "使用 Worktree 配置数据库分支"
description: "使用 Neon 或 PlanetScale 数据库分支进行隔离功能开发的指南"
tags: [workflow, git, devops]
---

# 使用 Worktree 配置数据库分支

隔离功能开发与数据库分支的完整指南。

**来源**：参考 [Neon 数据库分支](https://neon.com/docs/introduction/branching) 和 [PlanetScale 分支工作流](https://planetscale.com/docs/concepts/branching)。

---

## 简要说明（90% 的使用场景）

**使用 Neon：**
```bash
/git-worktree feature/auth
cd .worktrees/feature-auth
neonctl branches create --name feature-auth --parent main
# 将输出中的 DATABASE_URL 复制到 .env
pnpm prisma migrate dev
```

完成。直接跳到[工作流示例](#工作流示例)。

---

## 服务商配置

### Neon（推荐）

**安装 CLI：**
```bash
npm install -g neonctl
neonctl auth
```

**创建分支：**
```bash
neonctl branches create --name <branch-name> --parent main
```

**获取连接字符串：**
```bash
neonctl connection-string --branch <branch-name>
```

**更新 worktree 中的 .env：**
```bash
echo "DATABASE_URL=<connection-string>" > .worktrees/<branch>/.env
```

**完成后删除：**
```bash
neonctl branches delete <branch-name>
```

**优势：**
- 即时创建分支（约 1 秒）
- 真正的写时复制（高效存储）
- 分支重置无数据丢失
- 优秀的 CLI

**局限：**
- 需要配置连接池

---

### PlanetScale

**安装 CLI：**
```bash
brew install pscale
pscale auth login
```

**创建分支：**
```bash
pscale branch create <database-name> <branch-name>
```

**连接（启动本地代理）：**
```bash
pscale connect <database-name> <branch-name> --port 3309
```

**更新 .env 使用 localhost:3309：**
```bash
echo "DATABASE_URL=mysql://root@127.0.0.1:3309/<database-name>" > .worktrees/<branch>/.env
```

**完成后删除：**
```bash
pscale branch delete <database-name> <branch-name>
```

**优势：**
- 类似 Git 的 schema 工作流
- 内置 schema 差异对比
- 安全的部署请求

**局限：**
- 每个分支连接字符串不同
- 本地开发需要 `pscale connect`

---

### 本地 Postgres（基于 Schema）

适用于没有云数据库的项目：

**创建 schema：**
```bash
psql $DATABASE_URL -c "CREATE SCHEMA <schema-name>;"
```

**更新 .env 使用 schema：**
```bash
DATABASE_URL="postgresql://user:pass@localhost:5432/db?schema=<schema-name>"
```

**在 schema 中运行迁移：**
```bash
npx prisma migrate deploy
```

**清理：**
```bash
psql $DATABASE_URL -c "DROP SCHEMA <schema-name> CASCADE;"
```

**优势：**
- 免费
- 完全可控

**局限：**
- 手动配置
- 无自动写时复制

---

## 何时使用数据库分支

### 决策树

```
功能是否涉及数据库 schema？
├─ 否 → 使用共享数据库，跳过分支创建
└─ 是 → 创建数据库分支
    ├─ 使用 Neon/PlanetScale？ → 使用原生分支功能
    ├─ 使用本地 Postgres？ → 创建专用 schema
    └─ 其他服务商？ → 考虑 Docker 或谨慎使用共享数据库
```

### 场景对照表

| 场景 | 使用数据库分支？ | 理由 |
|------|----------------|------|
| 添加数据库迁移 | ✅ 是 | 隔离 schema 变更 |
| 重构数据模型 | ✅ 是 | 安全实验 |
| 性能测试 | ✅ 是 | 独立资源 |
| Bug 修复（无 schema 变更） | ❌ 否 | 共享数据库即可 |
| 带 schema 变更的功能 | ✅ 是 | 避免冲突 |
| 紧急热修复 | ❌ 否 | 速度优先于隔离 |

---

## 工作流示例

### 示例 1：Schema 迁移功能

```bash
# 1. 创建 worktree + 数据库分支
/git-worktree feature/add-user-roles
cd .worktrees/feature-add-user-roles

# 2. 创建 Neon 分支
neonctl branches create --name feature-add-user-roles --parent main

# 3. 用新的 DATABASE_URL 更新 .env
# （从 neonctl 输出中复制）

# 4. 分步创建迁移
npx prisma migrate dev --name step1_add_role_column
npx prisma migrate dev --name step2_migrate_existing_users
npx prisma migrate dev --name step3_add_constraints

# 5. 测试完整迁移序列
pnpm prisma migrate reset --skip-seed
pnpm prisma migrate deploy
pnpm test

# 6. 成功后合并 PR
# 7. 部署后应用到主数据库
```

---

### 示例 2：数据模型实验

```bash
# 无需承诺即可尝试不同 schema
/git-worktree experiment/normalize-addresses
cd .worktrees/experiment-normalize-addresses

# 创建数据库分支
neonctl branches create --name experiment-normalize-addresses --parent main

# 完全重塑数据
# 用近似真实的数据测试
# 对比性能

# 效果更好 → 合并
# 效果更差 → 删除分支（无需清理）
```

---

### 示例 3：并行功能开发

```bash
# 终端 1
/git-worktree feature/payments
cd .worktrees/feature-payments
neonctl branches create --name feature-payments --parent main
# DATABASE_URL → feature-payments 分支

# 终端 2
/git-worktree feature/subscriptions
cd .worktrees/feature-subscriptions
neonctl branches create --name feature-subscriptions --parent main
# DATABASE_URL → feature-subscriptions 分支

# 两者可独立修改 schema
# 合并前互不冲突
```

---

## 检查清单

### 在 worktree 中开始数据库变更前：
- [ ] `.worktreeinclude` 包含 `.env`
- [ ] 已创建数据库分支（如服务商支持）
- [ ] worktree 中的 `.env` 已更新为新的 `DATABASE_URL`
- [ ] 已测试连接（`npx prisma db execute --stdin <<< "SELECT 1;"`）
- [ ] 已应用迁移（`npx prisma migrate dev`）

### PR 合并后：
- [ ] 已移除 Git worktree
- [ ] 已删除数据库分支
- [ ] 无孤立连接

---

## 故障排查

### 问题：worktree 中出现"Database not found"
**修复：** 检查 `.env` 是否已复制，验证 `.worktreeinclude` 配置

### 问题：迁移影响了主数据库
**修复：** 确认 `DATABASE_URL` 指向分支而非主库

### 问题：无法创建 Neon 分支——"not authenticated"
**修复：** 运行 `neonctl auth` 登录

### 问题：PlanetScale 分支存在但无法连接
**修复：** 使用 `pscale connect` 代理，不要直接连接

### 问题："Branch already exists"
```bash
# 列出现有分支
neonctl branches list

# 如已过期则删除
neonctl branches delete <branch-name> --force
```

### 问题：迁移失败
```bash
# 将数据库分支重置为干净状态
neonctl branches reset <branch-name> --parent main

# 重新应用迁移
npx prisma migrate deploy
```

---

## 安全注意事项

⚠️ **注意：**
- 数据库分支默认不在 `.gitignore` 中
- 将 `.env` 添加到 `.worktreeinclude` 以便凭据被复制
- 切勿提交包含真实凭据的 `DATABASE_URL`
- 每个环境使用不同的凭据

✅ **最佳实践：**
```bash
# .worktreeinclude
.env
.env.local
.env.development

# 每个 worktree 获得凭据副本
# 但每个都指向不同的数据库分支
```

---

## 高级模式

### 模式：渐进式 Schema 迁移

```bash
# 1. 创建 worktree + 数据库分支
/git-worktree migration/split-user-table
cd .worktrees/migration-split-user-table

# 2. 分步创建迁移
npx prisma migrate dev --name step1_add_new_columns
npx prisma migrate dev --name step2_migrate_data
npx prisma migrate dev --name step3_drop_old_columns

# 3. 测试完整迁移序列
pnpm prisma migrate reset --skip-seed
pnpm prisma migrate deploy

# 4. 成功后合并 PR
# 5. 部署后应用到主数据库
```

### 模式：性能基准测试

```bash
# 创建带独立数据库的 worktree
/git-worktree perf/optimize-queries
cd .worktrees/perf-optimize-queries

# 数据库分支让你可以：
# - 添加索引而不影响开发环境
# - 安全地运行负载测试
# - 对比前后指标

# 仅合并经过验证的优化
```

---

**相关指南：**
- [Git Worktree 命令参考](../commands/git-worktree.md)
- [Neon 分支文档](https://neon.com/docs/introduction/branching)
- [PlanetScale 分支](https://planetscale.com/docs/concepts/branching)
