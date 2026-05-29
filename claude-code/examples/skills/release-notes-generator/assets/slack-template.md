> 📚 **AI Spark Wiki** · Claude Code 知识库

# Slack 公告模板

使用此模板生成以产品为导向的 Slack 消息。

```
Version X.Y.Z - Deployed to production

PR : [发布 PR 链接，例如：https://github.com/{owner}/{repo}/pull/XXX]

In brief : [用一句话描述本次发布的主要目标]

---

New Features

[功能名称 1]
> [1-2 句描述对用户的影响]
> Who is affected: [终端用户 / 管理员 / 所有人]

[功能名称 2]
> [1-2 句描述]
> Who is affected: [角色]

---

Important Fixes

- [用用户友好的语言描述缺陷修复]
- [用用户友好的语言描述缺陷修复]

---

Improvements

- [用用户语言描述 UX/UI 或工作流改进]
- [...]

---

By the numbers

- X 项新功能
- Y 个缺陷修复
- Z 项改进

---

Questions? Contact @[team-lead] or the dev team
```

## 指导原则

### 应该做
- 使用易懂的语言（避免技术术语）
- 聚焦于对用户的影响
- 简洁（每节最多 10 行）
- 适度使用 emoji

### 避免写
- "实现 DataLoader 模式以解决 N+1 查询"
- "使用 scope ANY/ASSIGNED 对权限系统进行完整重构"
- "从 webpack 迁移到 Turbopack"

### 技术语言 → 产品语言转换

| 技术表述 | 产品表述 |
|---------|---------|
| 使用 DataLoader 优化 N+1 查询 | 更快加载用户和功能列表 |
| 基于 pgvector 实现 AI 向量搜索 | 全新智能相似内容搜索 |
| 修复 getPermissionScope() 中的 scope 权限 | 修复了部分用户无法访问其数据的问题 |
| 将文件迁移到 kebab-case 命名 | *无需对外沟通（纯技术变更）* |
| 添加重试逻辑修复数据库连接 | 更稳定的数据库连接 |
| 为孤立记录添加错误监控 | 自动检测孤立记录 |

### 可选章节

如果本次发布包含重要信息，可添加：

Heads up
- [用户需要了解的重要信息]
- [他们可能注意到的行为变化]

Coming soon
- [下一个重要功能的预告]
- [即将上线的开发中功能]
