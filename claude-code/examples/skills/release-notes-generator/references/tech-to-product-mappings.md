> 📚 **AI Spark Wiki** · Claude Code 知识库

# 技术语言 → 产品语言转换规则

本文档定义了如何将技术性提交消息转换为用户友好的产品语言。

## 转换类别

### 1. 需要对外沟通（转换为产品语言）

| 技术模式 | 产品消息 |
|---------|---------|
| `N+1 queries`、`DataLoader`、`batching` | "列表加载速度更快" |
| `embeddings`、`vector search`、`pgvector` | "智能搜索能力提升" |
| `permissions`、`scope`、`access control` | "修复了一个访问权限问题" |
| `retry logic`、`resilience`、`connection errors` | "连接更加稳定" |
| `SSE`、`real-time`、`WebSocket` | "实时更新" |
| `cache`、`memoization` | "性能改善" |
| `responsive`、`mobile` | "移动端体验优化" |
| `accessibility`、`a11y`、`WCAG` | "无障碍访问改善" |
| `monitoring`、`alerting`、`error tracking` | "错误追踪能力增强" |
| `validation`、`sanitization` | "安全性增强" |

### 2. 无需对外沟通（仅限内部/技术层面）

以下模式**不应**出现在 Slack 公告中：

| 技术模式 | 原因 |
|---------|------|
| `refactor`、`refactoring` | 内部代码质量 |
| `webpack`、`turbopack`、`bundler` | 构建工具 |
| `eslint`、`prettier`、`linting` | 代码风格 |
| `kebab-case`、`naming convention` | 内部规范 |
| `TypeScript`、`type safety` | 开发者体验 |
| `test`、`spec`、`coverage` | 测试基础设施 |
| `chore`、`maintenance` | 日常维护 |
| `docs`、`documentation` | 内部文档 |
| `deps`、`dependencies`、`bump` | 依赖更新 |
| `CI`、`CD`、`workflow` | DevOps 基础设施 |

### 3. 安全相关（始终对外沟通，但要简化表述）

| 技术表述 | 产品表述 |
|---------|---------|
| `CVE-XXXX-XXXXX` | "修复了一个安全漏洞" |
| `XSS`、`injection` | "数据保护增强" |
| `authentication`、`auth bypass` | "登录安全性改善" |
| `CORS`、`CSRF` | "针对网络攻击的防护加强" |

## 场景化转换

### API 相关
- "Fix endpoint rate limiting" -> "API 稳定性改善"
- "Add request validation" -> "输入处理更完善"
- "Optimize query performance" -> "数据加载速度更快"

### 仪表盘相关
- "Fix dashboard widget rendering" -> "仪表盘显示问题修复"
- "Add export functionality" -> "新增数据导出功能"
- "Improve chart performance" -> "仪表盘加载速度更快"

### 通知相关
- "Fix email delivery queue" -> "通知可靠性提升"
- "Add webhook retry logic" -> "集成对接更稳定"
- "Optimize notification batching" -> "通知推送更及时"

### 搜索相关
- "Fix search indexing race condition" -> "搜索稳定性改善"
- "Add fuzzy matching" -> "搜索结果更精准"
- "Optimize search query execution" -> "搜索速度更快"

## 影响范围（按角色）

始终注明受影响的角色：

| 影响范围 | 角色 |
|---------|------|
| 仪表盘变更 | 终端用户、管理员 |
| API 变更 | 终端用户、高级用户 |
| 管理员面板 | 仅限管理员 |
| 计费/支付 | 管理员、利益相关方 |
| 报表/分析 | 管理员、高级用户 |
| 通知 | 所有用户 |
| 搜索 | 所有用户 |

## 严重级别标识

适当时使用以下前缀：

- **Critical（紧急）**：生产阻断性问题
- **Important（重要）**：面向用户的缺陷
- **Minor（次要）**：体验质量改善
- *不提及*：内部修复
