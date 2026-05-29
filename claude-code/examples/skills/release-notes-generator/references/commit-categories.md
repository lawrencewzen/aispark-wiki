> 📚 **AI Spark Wiki** · Claude Code 知识库

# 提交分类规则

本文档定义了如何基于 Conventional Commits 格式对提交进行分类。

## 主要分类

### 新功能（`feat:`）
**CHANGELOG**：新功能
**Slack**：是——始终包含
**示例**：
- `feat(dashboard): add export report system`
- `feat(search): add fuzzy matching`
- `feat(api): add batch operations endpoint`

### Bug 修复（`fix:`）
**CHANGELOG**：Bug 修复
**Slack**：是——如果影响用户；否——如果是内部修复
**示例**：
- `fix(auth): correct token refresh flow` -> 包含在 Slack 中
- `fix(test): correct mock setup` -> 不包含在 Slack 中

### 性能优化（`perf:`）
**CHANGELOG**：技术改进 > 性能
**Slack**：是——简化描述（"性能改进"）
**示例**：
- `perf(api): optimize N+1 queries with batching`
- `perf(build): reduce bundle size by 30%`

### 安全修复（`security:` 或 `fix(security):`）
**CHANGELOG**：安全
**Slack**：是——始终包含，并附适当的详细程度
**示例**：
- `security: fix CVE-2025-55182 dependency RCE`
- `fix(security): prevent XSS in user input`

## 次要分类（仅 CHANGELOG）

### 代码重构（`refactor:`）
**CHANGELOG**：技术改进 > 架构
**Slack**：否
**示例**：
- `refactor(hooks): migrate to new pattern`
- `refactor(permissions): extract to service layer`

### 文档（`docs:`）
**CHANGELOG**：文档（如果内容重要）
**Slack**：否
**示例**：
- `docs: update CLAUDE.md with new patterns`
- `docs(api): add endpoint documentation`

### 测试（`test:`）
**CHANGELOG**：测试（仅计数）
**Slack**：否
**示例**：
- `test(api): add endpoint integration tests`
- `test(e2e): add workflow tests`

### 日常维护（`chore:`）
**CHANGELOG**：否（除非内容重要）
**Slack**：否
**示例**：
- `chore: update dependencies`
- `chore(ci): fix workflow permissions`

### 样式（`style:`）
**CHANGELOG**：否
**Slack**：否
**示例**：
- `style: apply prettier formatting`
- `style(eslint): fix linting errors`

## Scope 模式

常用 scope：

| Scope | 领域 |
|-------|------|
| `auth` | 认证 |
| `billing` | 计费与支付 |
| `api` | API 端点 |
| `ui` | UI 组件 |
| `dashboard` | 仪表板功能 |
| `notifications` | 通知系统 |
| `search` | 搜索功能 |
| `user` | 用户管理 |
| `db` | 数据库与迁移 |
| `permissions` | 权限系统 |
| `admin` | 管理后台 |

## 破坏性变更

通过 type/scope 后的 `!` 或 footer 中的 `BREAKING CHANGE:` 标注：
- `feat(api)!: change status enum`
- `fix(auth)!: require new token format`

**CHANGELOG**：破坏性变更章节
**Slack**：是——附迁移说明

## PR 编号提取

从以下位置提取 PR 编号：
1. 提交信息：`(#123)`
2. Merge commit：`Merge pull request #123`
3. GitHub API：与提交 SHA 交叉引用

## 错误追踪器 Issue 关联

匹配以下模式：
- `[error-tracker]: PROJECT-XX`
- `fixes PROJECT-XX`
- `closes #XX`（GitHub issue）

## 统计数据计算

发布统计计数项：
- **PR 数**：唯一 PR 编号
- **功能数**：`feat:` 提交
- **Bug 修复数**：`fix:` 提交（排除测试/内部修复）
- **改进数**：`perf:` + `refactor:` + UI 改进
- **安全修复数**：`security:` 提交
- **破坏性变更数**：包含 `!` 或 `BREAKING CHANGE` 的提交
