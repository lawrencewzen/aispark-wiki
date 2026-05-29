> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: release-notes
description: 从 git 提交生成多种格式的发版说明
argument-hint: "[--from <tag>] [--format md|slack]"
effort: medium
disable-model-invocation: true
---

# 发版说明生成器

从 git 提交生成 3 种格式的发版说明，用于生产环境发布。

## 流程

1. **分析 Git 历史**：扫描自上次发布标签以来的提交
2. **获取 PR 详情**：通过 `gh api` 获取标题和描述
3. **分类变更**：按类型分组（feat、fix、perf 等）
4. **检查迁移**：检测数据库迁移文件
5. **生成 3 种输出**：CHANGELOG、PR 正文、通知消息
6. **转换语言**：将技术术语转换为产品语言

## 输出格式

### 1. CHANGELOG.md 章节

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Summary
[本次发布的 1-2 句概述]

### New Features
#### [Feature Name] (#PR)
- **Description**: 新增的面向用户的功能
- **Impact**: 如何使用户受益

### Bug Fixes
- **[Module]**: 描述 (#issue, tracking-ID)

### Technical Improvements
- [内部改进、重构、性能优化]

### Database Migrations
[如适用 - 列出迁移文件]

### Statistics
- PRs: X | Features: Y | Fixes: Z | Files changed: N
```

### 2. PR 发布正文

使用项目的发布模板：
- `.github/PULL_REQUEST_TEMPLATE/release.md`
- `.github/pull_request_template_release.md`
- 或项目配置中指定的自定义位置

### 3. 通知公告

生成面向用户的公告（Slack、邮件等）：
- 非技术性语言
- 聚焦于用户影响
- 可读性强的格式（可选 emoji）

模板位置示例：
- `.github/COMMUNICATION_TEMPLATE/slack-release.md`
- `docs/templates/release-announcement.md`

## 迁移警告

**如果检测到迁移：**

```
╔══════════════════════════════════════════════════════════════════╗
║  ⚠️  [ATTENTION] DATABASE MIGRATIONS REQUIRED                    ║
╠══════════════════════════════════════════════════════════════════╣
║  This release contains X migration(s):                           ║
║  • 20250110_add_user_preferences                                 ║
║  • 20250112_create_audit_log_table                               ║
║  Action required: Run migration command after deployment         ║
╚══════════════════════════════════════════════════════════════════╝
```

**如果没有迁移：**
```
✅ [OK] No database migrations required
```

## 技术语言转产品语言

将技术提交转换为用户友好的描述：

| 技术语言 | 产品/用户语言 |
|-----------|----------------------|
| "Optimize N+1 queries with DataLoader" | "列表加载速度更快" |
| "Implement AI embeddings with pgvector" | "全新智能搜索功能" |
| "Fix permissions scope bug" | "修复了部分用户的访问问题" |
| "Migration webpack -> Turbopack" | *仅内部 - 不对外传达* |
| "Refactor React hooks architecture" | *仅内部 - 不对外传达* |
| "Add rate limiting to API endpoints" | "提升系统稳定性与安全性" |

## 提交分类

| 前缀 | 类别 | 是否包含在公告中？ |
|--------|----------|--------------------------|
| `feat:` | 新功能 | 是 |
| `fix:` | Bug 修复 | 是（如面向用户） |
| `perf:` | 性能优化 | 是（简化描述） |
| `security:` | 安全 | 是 |
| `refactor:` | 架构 | 否 |
| `chore:` | 维护 | 否 |
| `docs:` | 文档 | 否 |
| `test:` | 测试 | 否 |
| `build:` | 构建系统 | 否 |
| `ci:` | CI/CD | 否 |

## 执行命令

```bash
# 1. 获取最新发布标签
LAST_TAG=$(git tag --sort=-v:refname | grep -E '^v[0-9]+\.[0-9]+\.[0-9]+$' | head -n 1)

# 2. 列出标签之后的提交（排除合并提交）
git log $LAST_TAG..HEAD --oneline --no-merges

# 3. 获取带 PR 编号的提交详情
git log $LAST_TAG..HEAD --format="%h %s" --no-merges

# 4. 检查迁移（根据 ORM 调整路径）
# Prisma:
git diff $LAST_TAG..HEAD --name-only -- prisma/migrations/
# Sequelize:
git diff $LAST_TAG..HEAD --name-only -- migrations/
# Django:
git diff $LAST_TAG..HEAD --name-only -- '**/migrations/*.py'
# Alembic:
git diff $LAST_TAG..HEAD --name-only -- alembic/versions/

# 5. 通过 GitHub CLI 获取 PR 详情
gh api repos/{owner}/{repo}/pulls/{number}

# 6. 统计数据
TOTAL_PRS=$(git log $LAST_TAG..HEAD --oneline --merges | wc -l)
FEATURES=$(git log $LAST_TAG..HEAD --oneline --no-merges | grep -c 'feat:')
FIXES=$(git log $LAST_TAG..HEAD --oneline --no-merges | grep -c 'fix:')
```

## 语义化版本

根据变更类型确定版本号：

| 变更类型 | 版本升级 | 示例 |
|-------------|--------------|---------|
| 破坏性变更 | MAJOR (X.0.0) | API 删除、不兼容变更 |
| 新功能 | MINOR (0.X.0) | 新功能、向后兼容 |
| Bug 修复/补丁 | PATCH (0.0.X) | 仅 Bug 修复 |

**判断依据**：
- 提交正文包含 `BREAKING CHANGE:` → MAJOR
- 存在 `feat:` 提交 → MINOR
- 仅有 `fix:` / `perf:` → PATCH

## 工作流集成

典型发布工作流：

```
1. 确认所有 PR 已合并到 develop 分支
2. 运行：/release-notes（或指定版本/范围）
3. 检查生成的输出是否准确
4. 创建 PR：develop -> main，带 "release" 标签
5. 将生成的 CHANGELOG 章节添加到 CHANGELOG.md
6. 使用生成的 PR 正文作为 PR 描述
7. 合并后：创建并推送 git 标签
8. 发布通知公告（Slack/邮件等）
9. 监控部署和迁移
```

## Git 标签创建

PR 合并后，创建带注释的标签：

```bash
# 创建带注释的标签
git tag -a v1.2.3 -m "Release v1.2.3: Brief description"

# 推送标签到远端
git push origin v1.2.3

# 或推送所有标签
git push --tags
```

## 项目专属定制

根据项目调整以下路径：

```
# 迁移检测（调整 ORM 路径）
prisma/migrations/        → 你的 ORM 迁移目录
db/migrate/               → Rails 迁移
alembic/versions/         → Alembic 迁移

# 模板文件（如需创建）
.github/PULL_REQUEST_TEMPLATE/release.md
.github/COMMUNICATION_TEMPLATE/announcement.md
docs/templates/release-notes.md
```

## 使用技巧

- **从仓库根目录运行**：确保 git 命令正常工作
- **认证 GitHub CLI**：如需运行 `gh auth login`
- **发布前审核**：始终验证生成的内容
- **破坏性变更**：在提交信息中搜索 `BREAKING CHANGE:`
- **关联 Issue**：包含 issue/ticket 编号以便追溯
- **数据库迁移**：上生产前先在 staging 环境测试

## 边界情况

| 场景 | 处理方式 |
|----------|----------|
| 未找到标签 | 从第一次提交开始 |
| 自上次标签无新提交 | 报错："No changes to release" |
| 同一提交上有多个标签 | 使用日期最新的 |
| 预发布标签（v1.0.0-beta.1） | 排除在"上次发布"搜索之外 |
| 不符合规范格式的提交 | 归类为"其他变更" |

## 使用示例

```bash
# 从最新标签到 HEAD 生成发版说明
/release-notes

# 手动指定版本
/release-notes v1.5.0

# 指定范围
/release-notes from v1.4.0 to HEAD

# 预览而不创建文件
/release-notes --preview

# 包含预发布提交
/release-notes --include-pre-release
```

Version/Range: $ARGUMENTS
