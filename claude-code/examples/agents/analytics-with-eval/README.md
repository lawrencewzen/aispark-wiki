> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "带内置评估的分析智能体"
description: "生产就绪的分析智能体，具备自动指标收集和安全验证功能"
tags: [agents, template, testing, performance]
---

# 带内置评估的分析智能体

**模板**：生产就绪的分析智能体，具备自动指标收集功能

**使用场景**：SQL 查询生成，配合质量追踪、性能监控和安全验证

**相关指南**：[智能体评估](../../../guide/roles/agent-evaluation.md)

---

## 包含内容

| 文件 | 用途 |
|------|---------|
| `analytics-agent.md` | 含评估标准的智能体定义 |
| `hooks/post-response-metrics.sh` | 每次响应后自动记录指标的钩子 |
| `eval/metrics.sh` | 用于汇总已收集指标的分析脚本 |
| `eval/report-template.md` | 月度评估报告模板 |

---

## 配置

### 1. 将智能体复制到项目

```bash
# 复制智能体定义
cp analytics-agent.md ~/.claude/agents/

# 或用于特定项目：
cp analytics-agent.md /path/to/project/.claude/agents/
```

### 2. 安装钩子

```bash
# 将钩子复制到项目
cp hooks/post-response-metrics.sh /path/to/project/.claude/hooks/

# 赋予执行权限
chmod +x /path/to/project/.claude/hooks/post-response-metrics.sh
```

### 3. 配置钩子触发器

添加到 `.claude/settings.json`：

```json
{
  "hooks": {
    "postToolUse": [
      {
        "command": ".claude/hooks/post-response-metrics.sh",
        "enabled": true,
        "description": "Log analytics agent metrics"
      }
    ]
  }
}
```

### 4. 创建日志目录

```bash
mkdir -p /path/to/project/.claude/logs
```

---

## 使用方法

### 调用智能体

```bash
# 在 Claude Code 会话中
"Use analytics-agent to generate a SQL query for [task description]"
```

### 查看指标

```bash
# 查看原始指标日志
cat .claude/logs/analytics-metrics.jsonl

# 运行分析
./examples/agents/analytics-with-eval/eval/metrics.sh
```

### 生成月度报告

```bash
# 复制模板
cp eval/report-template.md reports/analytics-2026-02.md

# 填入 metrics.sh 输出的指标数据
```

---

## 收集的指标

钩子自动记录：

| 指标 | 描述 | 来源 |
|--------|-------------|--------|
| `timestamp` | ISO 8601 时间戳 | 系统 |
| `query` | 生成的 SQL 查询 | 智能体响应 |
| `exec_time` | 查询执行时间 | 数据库 |
| `safety` | 破坏性操作的 PASS/FAIL | 查询分析 |
| `row_count` | 返回的行数 | 数据库 |
| `error` | 查询失败时的错误信息 | 数据库 |

**日志格式**：JSONL（JSON Lines），存储于 `.claude/logs/analytics-metrics.jsonl`

---

## 指标输出示例

```bash
$ ./eval/metrics.sh

=== Analytics Agent Metrics Report ===
Period: 2026-02-01 to 2026-02-10

Total queries: 45
Safety checks:
  - PASS: 42 (93%)
  - FAIL: 3 (7%)

Execution time:
  - Mean: 2.3s
  - Median: 1.8s
  - P95: 5.2s
  - P99: 8.1s

Common failures:
  1. DELETE without WHERE clause (2 occurrences)
  2. DROP TABLE in query (1 occurrence)

Recommendations:
  - Review agent instructions to emphasize WHERE clause requirement
  - Add explicit prohibition on DROP operations
```

---

## 定制

### 修改安全检查

编辑 `hooks/post-response-metrics.sh` 第 12-16 行：

```bash
# 添加更多匹配模式
if echo "$QUERY" | grep -iE 'DELETE|DROP|TRUNCATE|ALTER'; then
  SAFETY="FAIL"
fi
```

### 添加自定义指标

扩展 JSON 日志结构：

```bash
echo "{
  \"timestamp\":\"$(date -Iseconds)\",
  \"query\":\"$QUERY\",
  \"your_metric\":\"$VALUE\"
}" >> .claude/logs/analytics-metrics.jsonl
```

### 更改日志位置

更新钩子脚本中的 `POST_RESPONSE_LOG` 变量。

---

## 故障排查

### 钩子未触发

**检查**：
1. 钩子具有执行权限：`ls -l .claude/hooks/*.sh`
2. 钩子在 settings.json 中已启用
3. 智能体名称匹配：`analytics-agent`（带连字符）

**调试**：
```bash
# 手动测试钩子
export CLAUDE_RESPONSE='{"content":"SELECT * FROM users;"}'
./.claude/hooks/post-response-metrics.sh
```

### 日志中无指标

**检查**：
1. 日志目录存在：`mkdir -p .claude/logs`
2. 写入权限：`touch .claude/logs/test.log`
3. 查询提取模式与 SQL 格式匹配

### 指标分析失败

**检查**：
1. 已安装 `jq`：`which jq`
2. 日志文件为有效 JSONL：`jq . .claude/logs/analytics-metrics.jsonl`

---

## 生产环境注意事项

### 性能

- 钩子每次响应增加约 50ms 开销
- 日志文件每条查询增长约 200 字节
- 每月轮转日志以防文件过大

### 隐私

- 日志包含实际 SQL 查询（可能含敏感数据）
- 添加到 `.gitignore`：`.claude/logs/`
- 考虑在钩子脚本中脱敏敏感字段

### 数据库连接

- 钩子需要数据库访问以测量 `exec_time`
- 在钩子脚本中配置连接，或使用环境变量
- 出于安全考虑，确保使用只读凭据

---

## 相关资源

- **[智能体评估指南](../../../guide/roles/agent-evaluation.md)**：完整评估方法论
- **[钩子文档](../../../guide/ultimate-guide.md#5-hooks)**：钩子系统参考
- **[nao Framework](https://github.com/getnao/nao/)**：生产级分析智能体框架（灵感来源）

---

## 许可证

与父仓库相同（MIT）

**有疑问？** 请在主仓库中提交 issue 或发起讨论。
