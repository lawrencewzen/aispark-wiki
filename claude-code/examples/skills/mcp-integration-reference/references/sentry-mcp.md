> 📚 **AI Spark Wiki** · Claude Code 知识库

# Sentry MCP Server 参考文档

Sentry MCP server 的参考文件。在进行任何 Sentry MCP 调用之前请先阅读本文档。其中包含查询语法、已知注意事项以及可减少调用失败率的可用示例。

> 这是一个模板参考文件。请将 Sentry 相关内容替换为你自己 MCP server 的对应内容。

---

## 可用工具

### `mcp__sentry-mcp__list_issues`

**用途**：从一个或多个项目中获取问题列表。

**关键参数**：
- `organization_slug`（必填）：你的 Sentry 组织 slug，不是组织名称。从 Sentry URL 中获取：`sentry.io/organizations/<slug>/`。
- `project_slug`（可选）：限定到单个项目。省略则跨所有项目获取。
- `query`（可选）：Sentry 搜索查询。支持 `is:unresolved`、`level:error`、`has:user`。默认值：`is:unresolved`。
- `limit`（可选）：返回的最大问题数。默认值：25。最大值：100。

**注意事项**：
- 组织和项目 slug 为小写、连字符分隔。不要使用显示名称。
- 不带 `query` 时，会获取包括 `resolved` 在内的所有状态。除非你明确需要已解决的问题，否则始终加上 `is:unresolved`。
- `limit` 上限为每次调用 100 条。对于大型组织，使用 `cursor` 进行分页（见下文）。

### `mcp__sentry-mcp__get_issue`

**用途**：获取单个问题的完整详情，包括最新事件和堆栈追踪。

**关键参数**：
- `issue_id`（必填）：Sentry 问题的数字 ID。从 `list_issues` 中获取。
- `organization_slug`（必填）：同上。

**注意事项**：
- 仅返回最近一次事件。如需获取特定事件，请改用 `get_event` 并传入事件 ID。
- 堆栈帧默认从最外层到最内层排序。最底部的帧通常是崩溃点。

### `mcp__sentry-mcp__get_event`

**用途**：获取问题内特定事件的完整详情。

**关键参数**：
- `event_id`（必填）：完整的 32 字符事件 ID（UUID 格式，不带连字符）。
- `organization_slug`（必填）：同上。
- `issue_id`（必填）：所属问题的 ID。

**注意事项**：
- 事件 ID 大小写不敏感，但 API 区分大小写。请使用小写。
- 在默认保留方案下，超过 90 天的事件可能不可用。

### `mcp__sentry-mcp__search_events`

**用途**：跨问题搜索原始事件。比 `list_issues` 慢，但支持全文查询。

**关键参数**：
- `query`（必填）：全文搜索。支持字段过滤器：`message:`、`level:`、`user.id:`、`url:`、`transaction:`。
- `project_slug`（可选）：限定到单个项目。对于项目众多的组织（性能原因），此参数必填。
- `start` / `end`（可选）：ISO 8601 时间戳。默认：最近 24 小时。
- `limit`（可选）：默认值：10。最大值：100。

**注意事项**：
- 全文搜索大小写不敏感，但字段过滤器为精确匹配。`level:ERROR` 会失败，请使用 `level:error`。
- 时间范围必须使用 ISO 8601 格式：`2026-04-10T00:00:00Z`。此处不支持 `now-24h` 等相对格式（相对范围请使用 `list_issues`）。
- 不带 `project_slug` 时，大型组织的 `search_events` 会超时。搜索事件时请始终按项目限定范围。

---

## 查询语法

### Sentry 搜索查询（用于 `list_issues` 和 `search_events` 的 `query` 参数）

```
is:unresolved                          # 仅未解决的问题
is:unresolved level:error              # 未解决的错误（不含警告）
is:unresolved has:user                 # 影响已识别用户的问题
is:unresolved times_seen:>100          # 高频问题
project:api-service is:unresolved      # 限定到单个项目
assigned:me is:unresolved              # 分配给当前用户的问题
!has:assignee is:unresolved            # 未分配的问题
```

### 分页

对于问题较多的组织，使用基于游标的分页：
1. 第一次调用：`list_issues(..., limit=100)` — 响应中包含 `cursor` 字段
2. 下一页：`list_issues(..., limit=100, cursor="<上次响应的 cursor>")`
3. 当响应中没有 `cursor` 字段或问题数量小于 limit 时停止

---

## 已知模式与排除项

分析问题时，以下模式通常属于噪音，除非明确要求，否则应从报告中排除：

| 模式 | 排除原因 |
|------|---------|
| `/static/` 或 `/assets/` 上的 `404` 错误 | 预期行为：部署后浏览器请求了旧的静态资源 URL |
| 前端包中的 `ChunkLoadError` | 通常由相同的部署时序问题引起，而非代码 bug |
| `ResizeObserver loop limit exceeded` | 浏览器级别警告，无法采取行动 |
| 健康检查接口错误 | 监控基础设施，非用户可见问题 |

如果你排除了某个问题，请在报告的"范围外"章节中明确说明。

---

## 可用示例

### 获取生产环境中排名靠前的未解决错误

```
list_issues(
  organization_slug="your-org",
  query="is:unresolved level:error",
  limit=50
)
```

### 获取特定项目过去一周的问题

```
list_issues(
  organization_slug="your-org",
  project_slug="api-service",
  query="is:unresolved",
  limit=100
)
```

### 获取特定问题的完整堆栈追踪

```
get_issue(
  organization_slug="your-org",
  issue_id="1234567"
)
```

---

## 适配本文档

将此模板 fork 用于其他 MCP（Datadog、PagerDuty、Linear 等）时：

1. 将文件中所有"Sentry"替换为你的 MCP server 名称
2. 将工具名称（`mcp__sentry-mcp__*`）替换为你的 MCP 的实际工具名称
3. 为每个工具记录 2-3 个最常见的参数错误
4. 添加你的 MCP 专用查询语法（SQL 方言、过滤器语法等）
5. 添加"已知模式与排除项"章节，列出你数据源中的噪音模式
6. 包含 3-5 个覆盖 80% 使用场景的可用示例

目标：读完本文档后，Claude 应该能做到零语法错误、零因参数格式错误导致的重试调用。
