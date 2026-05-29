> 📚 **AI Spark Wiki** · Claude Code 知识库

# CHANGELOG 章节模板

使用此模板生成 CHANGELOG.md 条目。

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Objective
[1-2 句话概述本次发布]

### New Features

#### [功能名称] (#PR_NUMBER)
- **Description** : [清晰的功能描述]
- **Spec link** : [规格文档链接（如有）]
- **Impacted components** : `component-a`, `service-b`, etc.
- **Impact** : [受影响人员：终端用户 / 管理员 / 所有人]

### Bug Fixes

#### [模块/组件] (#PR_NUMBER)
- **Issue** : [缺陷描述]
- **Cause** : [已定位的根本原因]
- **Fix** : [修复方案描述]
- **[Error tracker]** : PROJECT-XX（如适用）

### Technical Improvements

#### Performance
- [优化描述，尽量包含可量化的影响]

#### UI/UX
- [界面改进描述]

#### Architecture
- [重大重构描述]

### Security
- **[CVE-XXXX-XXXXX]** : [描述及影响]

### Database Migrations

#### Deployment Process

**Step 1: Apply migrations**
```bash
[migration-command]
```

**Step 2: Data migration scripts** (if applicable)
```bash
[data-migration-command]
```

**Post-migration verification**
```sql
-- Verification queries
SELECT COUNT(*) FROM [table];
```

### Breaking Changes

**None** or:

- **[Component/API]** : 破坏性变更描述
  - **Migration required** : 迁移方式
  - **Impact** : 受影响人员

### Deprecations

**None** or:

- **[Feature X]** : 本版本起废弃
  - **Reason** : 原因
  - **Alternative** : 替代方案
  - **Planned removal** : 计划在 X.Y.Z 版本移除

### Tests
- [X] unit tests for [feature]
- [X] integration tests for [module]

### Statistics
- **PRs** : #XX, #YY, #ZZ
- **Files impacted** : XX+
- **New tables** : [如有请列出]
- **Migrations** : X 条迁移
- **Breaking changes** : 0

### Links
- PR: https://github.com/{owner}/{repo}/pull/XXX
- Included PRs : #XX, #YY, #ZZ
- [Error tracker] issues : PROJECT-XX
```
