> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: release-notes-generator
description: "从 git 提交生成 3 种格式的发版说明（CHANGELOG.md、PR 正文、Slack 公告）。自动分类变更并将技术语言转换为用户友好的表述。适用于发版、变更日志、版本说明、新功能摘要或上线公告。"
allowed-tools: Bash
effort: low
---

# 发版说明生成器

从 git 提交生成 3 种格式的完整发版说明。

## 工作流

1. **分析 git 历史**：从最新发布标签或指定版本开始
2. **获取 PR 详情**：通过 `gh api` 获取标题、描述和标签
3. **分类变更**：归类为新功能、Bug 修复、改进、安全和破坏性变更
4. **生成 3 种输出**：CHANGELOG.md 章节（技术向）、PR 发布正文（半技术向）、Slack 消息（用户友好）
5. **转换语言**：将技术术语转为通俗易懂的表述
6. **迁移警告**：若检测到数据库迁移则发出提示

## 使用方法

### 基本用法

```
Generate release notes since last release
```

```
Create release notes for version 0.18.0
```

### 指定范围

```
Generate release notes from v0.17.0 to HEAD
```

### 仅预览

```
Preview release notes without writing files
```

## 输出格式

### 1. CHANGELOG.md 章节

面向开发者的技术格式：

```markdown
## [0.18.0] - 2025-12-08

### Objective
[1-2 句摘要]

### New Features
#### [Feature Name] (#PR)
- **Description**: ...
- **Impact**: ...

### Bug Fixes
- **[Module]**: 描述 (#issue, [error-tracker] ISSUE-XX)

### Technical Improvements
#### Performance / UI/UX / Architecture
- [描述]

### Database Migrations
[如适用]

### Statistics
- PRs: X
- Features: Y
- Bugs: Z
```

### 2. PR 发布正文

使用 `.github/PULL_REQUEST_TEMPLATE/release.md` 中的模板：
- 目标摘要
- 含规格链接的功能列表
- 含错误追踪引用的 Bug 修复
- 按类别划分的改进项
- 迁移说明
- 部署检查清单

### 3. Slack 公告

使用 `.github/COMMUNICATION_TEMPLATE/slack-release.md` 中的产品导向格式：
- 包含 **PR 链接**以便追溯
- 非技术性语言
- 聚焦于用户影响（终端用户、管理员、利益相关方）
- 使用 emoji 提升可读性
- 统计摘要

## 工作流集成

本技能与发布工作流集成：

```
1. 分析提交：git log <last-tag>..HEAD
2. 确定版本号（MAJOR.MINOR.PATCH）
3. 生成 3 种输出
4. 创建 PR：develop -> main，带 "Release" 标签
5. 更新 CHANGELOG.md
6. 合并后：创建 git 标签
7. 生成 Slack 公告
```

## 技术语言转产品语言

本技能自动转换技术语言：

| 技术语言 | 产品语言 |
|-----------|---------|
| "N+1 query optimization with DataLoader" | "列表加载更快" |
| "AI embeddings implementation with pgvector" | "全新智能搜索" |
| "Fix scope permissions in getPermissionScope()" | "修复访问权限问题" |
| "Migration webpack -> Turbopack" | *不对外传达* |
| "React hooks refactoring" | *不对外传达* |
| "Fix N+1 in user loaders" | "性能优化" |
| "Add retry logic for DB connection errors" | "连接更稳定" |

## 提交分类

提交按规范提交前缀进行分类：

| 前缀 | 类别 | 是否包含在 Slack 中？ |
|--------|----------|-------------------|
| `feat:` | 新功能 | 是 |
| `fix:` | Bug 修复 | 是（如面向用户） |
| `perf:` | 性能优化 | 是（简化描述） |
| `security:` | 安全 | 是 |
| `refactor:` | 架构 | 否 |
| `chore:` | 维护 | 否 |
| `docs:` | 文档 | 否 |
| `test:` | 测试 | 否 |
| `style:` | 样式 | 否 |

## 控制台输出格式

生成发版说明时，**务必**在控制台显示清晰的摘要，包含：

### 迁移警告（重要）

如果检测到迁移，突出显示：

```
╔══════════════════════════════════════════════════════════════════╗
║  ⚠️  [WARNING] DATABASE MIGRATIONS REQUIRED                      ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  This release contains X migration(s):                           ║
║                                                                  ║
║  • 20251210123456_add_feature_description                        ║
║  • 20251211094532_update_status_enum                             ║
║                                                                  ║
║  Action required BEFORE deployment:                              ║
║  → Run: [migration-command]                                      ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

如果**没有**迁移：

```
✅ [OK] No database migrations required
```

### 检测方法

检查自上次发布以来的新迁移文件：

```bash
# 查找自上次标签以来的新迁移
git diff <last-tag>..HEAD --name-only -- migrations/
```

## 示例

**用户**："Generate release notes since v0.17.0"

**输出**：

```
Analyzing commits from v0.17.0 to HEAD...

Found:
- 15 commits
- 8 PRs merged
- 2 new features
- 5 bug fixes
- 3 improvements

╔══════════════════════════════════════════════════════════════════╗
║  ⚠️  [WARNING] DATABASE MIGRATIONS REQUIRED                      ║
╠══════════════════════════════════════════════════════════════════╣
║  This release contains 1 migration(s):                           ║
║  • 20251208143021_add_user_preferences                           ║
║  Action required: [migration-command]                             ║
╚══════════════════════════════════════════════════════════════════╝

--- CHANGELOG.md Section ---
[技术格式输出]

--- PR Release Body ---
[半技术格式输出]

--- Slack Announcement ---
[产品导向格式输出]

Write to files? (CHANGELOG.md, clipboard for PR/Slack)
```

## 使用的命令

```bash
# 获取最新发布标签
git tag --sort=-v:refname | head -n 1

# 列出标签之后的提交
git log <tag>..HEAD --oneline --no-merges

# 获取 PR 详情
gh api repos/{owner}/{repo}/pulls/{number}

# 获取提交详情
git show --stat <sha>
```

## 使用技巧

- 从仓库根目录运行
- 确保 `gh` CLI 已认证
- 发布前审核生成的内容
- 根据受众调整产品语言
- 使用 `--preview` 预览输出而不写入文件

## 参考文件

- `assets/changelog-template.md` — CHANGELOG 章节模板
- `assets/slack-template.md` — Slack 公告模板
- `references/tech-to-product-mappings.md` — 语言转换规则
- `references/commit-categories.md` — 分类规则

## 相关技能

- `github-actions-templates` — 用于 CI/CD 工作流
- `changelog-generator` — 原始灵感来源（ComposioHQ）
