> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "会话可观测性与监控"
description: "跨开发会话追踪 Claude Code 使用量、估算成本并识别使用模式"
tags: [observability, guide, performance]
---

# 会话可观测性与监控

> 跨开发会话追踪 Claude Code 使用量、估算成本并识别使用模式。

## 目录

1. [为什么要监控会话](#为什么要监控会话)
2. [会话搜索与恢复](#会话搜索与恢复)
3. [设置会话日志记录](#设置会话日志记录)
4. [分析会话数据](#分析会话数据)
5. [成本追踪](#成本追踪)
6. [活动监控](#活动监控)
7. [外部监控工具](#外部监控工具)
8. [代理 Claude Code](#代理-claude-code)
9. [模式与最佳实践](#模式与最佳实践)
10. [局限性](#局限性)

---

## 为什么要监控会话

Claude Code 的使用量可能在活跃开发中快速积累。监控帮助你：

- **了解成本**：在账单到来之前估算 API 支出
- **识别模式**：查看哪些工具使用最多，哪些文件被反复编辑
- **优化工作流**：发现低效之处（例如反复读取同一个大文件）
- **跟踪项目**：比较不同代码库之间的使用情况
- **团队可见性**：聚合日志实现团队级预算管理

---

## 会话搜索与恢复

使用 Claude Code 数周后，查找过去的对话会变得困难。本节介绍原生选项和社区工具。

### 原生命令

| 命令 | 用例 |
|---------|----------|
| `claude -c` / `claude --continue` | 恢复最近的会话 |
| `claude -r <id>` / `claude --resume <id>` | 通过 ID 恢复特定会话 |
| `claude --resume` | 交互式会话选择器 |

会话以 JSONL 文件形式存储在 `~/.claude/projects/<project>/`。

### 社区工具对比

| 工具 | 安装 | 列表速度 | 搜索速度 | 依赖 | 恢复命令 |
|------|---------|------------|--------------|--------------|----------------|
| **session-search.sh**（本仓库） | 复制脚本 | **10ms** | **400ms** | 无（bash） | ✅ 显示 |
| claude-conversation-extractor | `pip install` | 230ms | 1.7s | Python | ❌ |
| claude-code-transcripts | `uvx` | N/A | N/A | Python | ❌ |
| ran CLI | `npm -g` | N/A | 快速 | Node.js | ❌（仅命令） |

### 推荐：session-search.sh

零依赖 bash 脚本，为速度优化，并显示可直接使用的恢复命令。

**安装：**
```bash
cp examples/scripts/session-search.sh ~/.claude/scripts/cs
chmod +x ~/.claude/scripts/cs
echo "alias cs='~/.claude/scripts/cs'" >> ~/.zshrc
source ~/.zshrc
```

**使用方法：**
```bash
cs                          # 列出最近 10 个会话（约 15ms）
cs "authentication"         # 单关键词搜索（约 400ms）
cs "Prisma migration"       # 多词 AND 搜索（两者都必须匹配）
cs -n 20                    # 显示 20 个结果
cs -p myproject "bug"       # 按项目名过滤
cs --since 7d               # 最近 7 天的会话
cs --since today            # 仅今天的会话
cs --json "api" | jq .      # JSON 输出用于脚本
cs --rebuild                # 强制重建索引
```

**输出：**
```
2026-01-15 08:32 │ my-project             │ 实现 OAuth 流程...
  claude --resume 84287c0d-8778-4a8d-abf1-eb2807e327a8

2026-01-14 21:13 │ other-project          │ 修复数据库迁移...
  claude --resume 1340c42e-eac5-4181-8407-cc76e1a76219
```

复制粘贴 `claude --resume` 命令可继续任意会话。

### 工作原理

1. **索引模式**（无过滤器）：使用缓存的 TSV 索引。会话变更时自动刷新。约 15ms 查找。
2. **搜索模式**（带关键词/过滤器）：3 秒超时的全文搜索。多词查询使用 AND 逻辑。
3. **过滤器**：`--project`（子串匹配）、`--since`（支持 `today`、`yesterday`、`7d`、`YYYY-MM-DD`）
4. **输出**：默认人类可读，`--json` 用于脚本。排除智能体/子智能体会话。

### 替代方案：Python 工具

如果你偏好更丰富的功能（HTML 导出、多种格式）：

```bash
# 安装
pip install claude-conversation-extractor

# 交互式 UI
claude-start

# 直接搜索
claude-search "keyword"

# 导出为 markdown
claude-extract --format markdown
```

完整脚本见 [session-search.sh](../../examples/scripts/session-search.sh)。

---

### 会话恢复限制与跨文件夹迁移

**TL;DR**：原生 `--resume` 因设计原因限于当前工作目录。跨文件夹迁移推荐使用手动文件系统操作，社区自动化工具未经充分测试。

#### 为什么恢复仅限于目录

Claude Code 将会话存储在 `~/.claude/projects/<encoded-path>/`，其中 `<encoded-path>` 由项目的绝对路径派生。例如：
- 项目在 `/home/user/myapp` → 会话在 `~/.claude/projects/-home-user-myapp-/`
- 项目移动到 `/home/user/projects/myapp` → Claude 查找 `~/.claude/projects/-home-user-projects-myapp-/`（不同目录）

**设计理由**：会话存储了绝对文件路径、项目特定上下文（MCP 服务器配置、`.claudeignore` 规则、环境变量）。跨文件夹恢复需要路径重写和上下文验证，目前尚未实现。

**相关**：GitHub issue [#1516](https://github.com/anthropics/claude-code/issues/1516) 追踪社区对原生跨文件夹支持的需求。

#### 手动迁移（推荐）

**移动项目文件夹时：**

```bash
# 移动项目前
cd ~/.claude/projects/
ls -la  # 记录当前编码路径

# 移动你的项目
mv /old/location/myapp /new/location/myapp

# 重命名会话目录以匹配新路径
cd ~/.claude/projects/
mv -- -old-location-myapp- -new-location-myapp-

# 验证
cd /new/location/myapp
claude --continue  # 应该成功恢复
```

**将会话分叉到新项目时：**

```bash
# 复制会话文件（保留原始）
cd ~/.claude/projects/
cp -n ./-source-project-/*.jsonl ./-target-project-/

# 如果存在则复制 subagents 目录
if [ -d ./-source-project-/subagents ]; then
  cp -r ./-source-project-/subagents ./-target-project-/
fi

# 在目标项目中恢复
cd /path/to/target/project
claude --continue
```

#### ⚠️ 迁移风险与注意事项

**迁移会话前，验证兼容性：**

| 风险 | 影响 | 缓解措施 |
|------|--------|------------|
| **硬编码的 secrets** | 凭证在新上下文中暴露 | 迁移前审计 `.jsonl` 文件，必要时脱敏 |
| **绝对路径** | 路径不同时文件引用失效 | 验证目标中路径存在，或接受失效引用 |
| **MCP 服务器配置** | 源 MCP 服务器在目标中缺失 | 恢复前安装匹配的 MCP 服务器 |
| **`.claudeignore` 规则** | 不同的忽略模式 | 检查差异，必要时合并 |
| **环境变量** | `process.env` 上下文不匹配 | 检查 `.env` 文件兼容性 |

**何时不迁移会话：**

- 依赖冲突（例如不同 Node.js 版本、包管理器）
- 数据库状态差异（源中应用了迁移，目标中没有）
- 认证上下文（特定于源项目的 API 令牌、OAuth 会话）
- 安全边界（从私有仓库迁移到公开仓库）

#### 社区自动化工具

**claude-migrate-session**（Jim Weller 编写，受 Alexis Laporte 启发）自动化上述手动过程：

- **仓库**：[jimweller/dotfiles](https://github.com/jimweller/dotfiles/tree/main/dotfiles/claude-code/skills/claude-migrate-session)
- **功能**：带过滤的全局搜索，保留 `.jsonl` + subagents，使用 ripgrep（rg）提高性能
- **状态**：个人 dotfiles（截至 2026 年 2 月 0 stars/forks），采用有限
- **命令**：`/claude-migrate-session <source> <target>`

**⚠️ 注意**：该工具社区测试极少。手动方法更安全，让你明确控制迁移内容。在生产工作流中使用前彻底测试。

**迁移使用场景：**
- 将原型工作分叉到生产代码库
- 将调试会话移动到隔离的测试仓库
- 在新项目中继续架构讨论

#### 替代方案：Entire CLI 会话可移植性

**原生限制**：Claude Code 的 `--resume` 与绝对文件路径绑定，文件夹移动后会失效。

**Entire CLI 解决方案**：检查点是**路径无关的**，实现跨项目位置的真正会话可移植性。

**工作原理：**

```bash
# 在源项目中
cd /old/location/myapp
entire capture --agent="claude-code"
[... 在 Claude Code 中工作 ...]
entire checkpoint --name="migration-complete"

# 将项目移动到新位置
mv /old/location/myapp /new/location/myapp

# 在目标中恢复（因为 Entire 存储相对路径，所以有效）
cd /new/location/myapp
entire resume --checkpoint="migration-complete"
claude --continue  # 带完整上下文恢复
```

**为什么 Entire 检查点可移植：**

| 方面 | 原生 `--resume` | Entire CLI |
|--------|-------------------|-----------|
| **路径存储** | JSONL 中的绝对路径 | 检查点中的相对路径 |
| **跨文件夹** | 失效（不同项目编码） | 有效（路径无关） |
| **上下文保存** | 仅提示词历史 | 提示词 + 推理 + 文件状态 |
| **智能体交接** | 否 | 是（Claude/Gemini 之间） |

**何时选择 Entire 而非手动迁移：**

- ✅ 频繁的项目移动/分叉
- ✅ 多智能体工作流（Claude → Gemini 交接）
- ✅ 调试的会话重放（回溯到精确状态）
- ✅ 治理（恢复时的审批门）

**权衡**：增加工具依赖 + 存储开销（约项目大小的 5-10%）。

> **完整文档**：[AI 可追溯性指南](./ai-traceability.md#51-entire-cli)

---

### 多智能体编排监控

对于通过外部编排器（Gas Town、multiclaude）监控多个并发 Claude Code 实例，参见：

- **agent-chat**（https://github.com/justinabrahms/agent-chat）：智能体通信的实时 Slack 式 UI
- **架构指南**：`guide/ai-ecosystem.md` 第 8.1 节 - 多智能体编排系统

**架构模式**（用于自定义实现）：
1. Hook 记录任务智能体生成：`.claude/hooks/multi-agent-logger.sh`
2. 存储到 SQLite：`~/.claude/logs/agents.db`（parent_id、child_id、timestamp、task）
3. 通过 SSE 流式传输：简单的 Go/Node HTTP 服务器
4. 仪表板：消费 SSE 流的 React/HTML

**原生 Claude Code 监控**（本指南）：
- 会话搜索：`session-search.sh`（见[会话搜索与恢复](#会话搜索与恢复)）
- 活动日志：`session-logger.sh` hook（见[设置会话日志记录](#设置会话日志记录)）
- 统计分析：`session-stats.sh`（见[分析会话数据](#分析会话数据)）

**何时使用外部编排器监控**：
- 运行带 5+ 并发智能体的 Gas Town 或 multiclaude
- 需要实时查看智能体协调情况
- 调试编排失败（智能体冲突、合并问题）

**何时原生监控已足够**：
- 单个 Claude Code 会话或 `--delegate` 带 <3 个子智能体
- 事后分析（日志、统计）已足够
- 预算/复杂度限制

---

## 设置会话日志记录

### 1. 安装 Logger Hook

将会话记录器复制到 hooks 目录：

```bash
# 如有必要创建 hooks 目录
mkdir -p ~/.claude/hooks

# 从本仓库示例复制记录器
cp examples/hooks/bash/session-logger.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/session-logger.sh
```

### 2. 在设置中注册

添加到 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "~/.claude/hooks/session-logger.sh"
      }
    ]
  }
}
```

### 3. 验证安装

运行几个 Claude Code 命令，然后检查日志：

```bash
ls ~/.claude/logs/
# 应该看到：activity-2026-01-14.jsonl

# 查看最近条目
tail -5 ~/.claude/logs/activity-$(date +%Y-%m-%d).jsonl | jq .
```

### 配置选项

| 环境变量 | 默认值 | 描述 |
|---------------------|---------|-------------|
| `CLAUDE_LOG_DIR` | `~/.claude/logs` | 日志存储位置 |
| `CLAUDE_LOG_TOKENS` | `true` | 启用 Token（词元）估算 |
| `CLAUDE_SESSION_ID` | 自动生成 | 自定义会话标识符 |

---

## 分析会话数据

### 使用 session-stats.sh

```bash
# 复制脚本
cp examples/scripts/session-stats.sh ~/.local/bin/
chmod +x ~/.local/bin/session-stats.sh

# 今日摘要
session-stats.sh

# 最近 7 天
session-stats.sh --range week

# 特定日期
session-stats.sh --date 2026-01-14

# 按项目过滤
session-stats.sh --project my-app

# 机器可读输出
session-stats.sh --json
```

### 示例输出

```
═══════════════════════════════════════════════════════════
        Claude Code 会话统计 - 今天
═══════════════════════════════════════════════════════════

摘要
  总操作数:  127
  会话数:    3

Token（词元）使用量
  输入 Token（词元）:      45,230
  输出 Token（词元）:     12,450
  总计 Token（词元）:      57,680

估算成本（Sonnet 费率）
  输入:   $0.1357
  输出:  $0.1868
  总计:   $0.3225

工具使用
  Edit:  45
  Read:  38
  Bash:  24
  Grep:  12
  Write: 8

项目
  my-app: 89
  other-project: 38
```

### 关注质量，不只是数量

Token（词元）计数告诉你使用了多少 Claude Code。JSONL 日志还可以告诉你**配置的效果如何**——如果你知道找什么。

除成本指标外，三种模式可靠地表明 Skill（技能模块）、规则或 CLAUDE.md 章节需要更新：

**重复读取相同文件**

如果 Claude 在一次会话中读取同一文件 3 次以上，它需要的内容可能不在预期位置。考虑将相关上下文移入 Skill（技能模块）或 CLAUDE.md 章节。

```bash
# 最近会话中被读取超过 3 次的文件
jq -r 'select(.tool == "Read") | .file' ~/.claude/logs/activity-*.jsonl \
  | sort | uniq -c | sort -rn | awk '$1 > 3'
```

**相同命令上的工具失败**

跨会话反复失败的 Bash 命令通常意味着 Skill（技能模块）的路径过时、二进制文件被重命名，或命令对当前技术栈不再有效。

```bash
# 失败的命令
jq -r 'select(.tool == "Bash" and (.exit_code // 0) != 0) | .command' \
  ~/.claude/logs/activity-*.jsonl | sort | uniq -c | sort -rn | head -10
```

**同一文件上的高编辑频率**

跨会话大量编辑的文件通常表明上下文缺失——智能体不清楚该文件的用途，或围绕它的约定未被记录。

```bash
# 最多编辑的文件（上下文缺口的代理指标）
jq -r 'select(.tool == "Edit") | .file' ~/.claude/logs/activity-*.jsonl \
  | sort | uniq -c | sort -rn | head -10
```

对你发现的每种模式，问：是否有 Skill（技能模块）、规则或 CLAUDE.md 章节应该覆盖这一点？完整工作流见 [§9.23 配置生命周期与更新循环](#923-configuration-lifecycle-the-update-loop)。

---

### 日志格式

每个日志条目是一个 JSON 对象：

```json
{
  "timestamp": "2026-01-14T15:30:00Z",
  "session_id": "1705234567-12345",
  "tool": "Edit",
  "file": "src/components/Button.tsx",
  "project": "my-app",
  "tokens": {
    "input": 350,
    "output": 120,
    "total": 470
  }
}
```

---

## 成本追踪

### Token（词元）估算方法

记录器使用简单启发式估算 Token（词元）：**约 4 个字符 = 1 个 Token（词元）**。这是近似值，倾向于轻微高估。

### 费率

默认费率适用于 Claude Sonnet。通过环境变量调整：

```bash
# Sonnet 费率（默认）
export CLAUDE_RATE_INPUT=0.003   # 3 美元/百万 Token（词元）
export CLAUDE_RATE_OUTPUT=0.015  # 15 美元/百万 Token（词元）

# Opus 费率（如果使用 Opus）
export CLAUDE_RATE_INPUT=0.015   # 15 美元/百万 Token（词元）
export CLAUDE_RATE_OUTPUT=0.075  # 75 美元/百万 Token（词元）

# Haiku 费率
export CLAUDE_RATE_INPUT=0.00025 # 0.25 美元/百万 Token（词元）
export CLAUDE_RATE_OUTPUT=0.00125 # 1.25 美元/百万 Token（词元）
```

### 预算告警（手动模式）

添加到 shell 配置文件以获得每日预算警告：

```bash
# ~/.zshrc 或 ~/.bashrc
claude_budget_check() {
  local cost=$(session-stats.sh --json 2>/dev/null | jq -r '.summary.estimated_cost.total // 0')
  local threshold=5.00  # 5 美元每日预算

  if (( $(echo "$cost > $threshold" | bc -l) )); then
    echo "⚠️  Claude Code 今日支出：\$$cost（阈值：\$$threshold）"
  fi
}

# shell 启动时运行
claude_budget_check
```

---

## 活动监控

成本追踪告诉你*花了多少*。活动监控告诉你 Claude Code *实际做了什么*：读了哪些文件、运行了哪些命令、获取了哪些 URL。这是审计层。

### 会话 JSONL：事实来源

Claude Code 进行的每次工具调用都记录在 `~/.claude/projects/<project>/` 的会话 JSONL 文件中。`type: "assistant"` 的每个条目包含一个 `content` 数组，其中 `type: "tool_use"` 块记录了每次操作。

```bash
# 查找你的会话文件
ls ~/.claude/projects/-$(pwd | tr '/' '-')-/

# 检查会话中的工具调用
cat ~/.claude/projects/-your-project-/SESSION_ID.jsonl | \
  jq 'select(.type == "assistant") | .message.content[]? | select(.type == "tool_use") | {tool: .name, input: .input}'
```

### 工具调用揭示的内容

| 工具 | 暴露内容 |
|------|----------------|
| `Read` | 访问的文件（路径、行范围） |
| `Write` / `Edit` | 修改的文件（路径、内容差异） |
| `Bash` | 执行的命令（完整命令字符串） |
| `WebFetch` | 获取的 URL（可能包含 POST 中发送的数据） |
| `Task` | 子智能体生成（传递给子模型的提示词） |
| `Glob` / `Grep` | 搜索模式和范围 |

### 实用审计查询

```bash
# 会话中读取的所有文件
SESSION=~/.claude/projects/-your-project-/SESSION_ID.jsonl
jq 'select(.type == "assistant") | .message.content[]? | select(.type == "tool_use" and .name == "Read") | .input.file_path' "$SESSION"

# 执行的所有 bash 命令
jq 'select(.type == "assistant") | .message.content[]? | select(.type == "tool_use" and .name == "Bash") | .input.command' "$SESSION"

# 获取的所有 URL
jq 'select(.type == "assistant") | .message.content[]? | select(.type == "tool_use" and .name == "WebFetch") | .input.url' "$SESSION"

# 按类型统计工具使用
jq -r 'select(.type == "assistant") | .message.content[]? | select(.type == "tool_use") | .name' "$SESSION" | sort | uniq -c | sort -rn
```

### 需要关注的敏感模式

以下工具调用模式值得在自动审计中标记：

| 模式 | 风险 | 检测方式 |
|---------|------|-----------|
| `Read` 读取 `.env`、`*.pem`、`id_rsa` | 凭证访问 | `jq '... | select(.input.file_path | test("\\.(env|pem|key)$"))'` |
| `Bash` 执行 `rm -rf`、`git push --force` | 破坏性操作 | `jq '... | select(.input.command | test("rm -rf\|force-push"))'` |
| `WebFetch` 访问外部 URL | 数据泄露风险 | `jq '... | select(.name == "WebFetch") | .input.url'` |
| `Write` 写入项目根目录外的文件 | 范围蔓延 | 对比工作目录检查路径 |

> **安全上下文**：Claude Code 以你的用户权限对文件系统进行读写操作。JSONL 审计追踪是你的操作记录。对于团队，考虑将这些日志同步到不可变存储。

---

## 外部监控工具

除上述基于 Hook 的方法外，社区还构建了专用工具。以下是截至 2026 年初的事实快照。

| 工具 | 类型 | 功能 | 安装 |
|------|------|-------------|---------|
| **ccusage** | CLI / TUI | 从 JSONL 追踪成本——定价数据的事实参考。约 10,000 GitHub stars。 | `npm i -g ccusage` |
| **claude-code-otel** | OpenTelemetry 导出器 | 向任意 OTEL 收集器发送 spans。与 Prometheus + Grafana 仪表板集成。面向企业。 | `npm i -g claude-code-otel` |
| **Akto** | SaaS / 自托管 | API 安全护栏 + 审计追踪。在 API 层拦截，标记违反策略的行为。 | [akto.io](https://akto.io) |
| **MLflow Tracing** | CLI + SDK | 精确 Token（词元）计数、工具 spans、LLM 作为评判者评估。CLI 模式：无需 Python。最适合 ML/MLOps 团队。 | `pip install mlflow` → [见下方章节](#mlflow-tracing) |
| **ccboard** | TUI + Web | 会话、成本、统计的统一仪表板。活动/审计标签正在开发中。 | `cargo install ccboard` |
| **claude-crusts** | CLI | 一键上下文污染扫描器：标记过时文件、过大记忆和冗余规则加载。在运行会话前找出膨胀上下文的原因。 | [github.com/Abinesh-L/claude-crusts](https://github.com/Abinesh-L/claude-crusts) |

### 决策指南

```
快速获取成本数字？          → ccusage（CLI，零配置）
需要企业级审计追踪？        → claude-code-otel + Grafana 或 Akto
已经在 ML 中使用 MLflow？   → MLflow 追踪集成（见下方）
需要智能体回归检测？        → MLflow 追踪 + LLM 作为评判者
需要持久化 TUI/Web UI？     → ccboard
上下文污染审计？            → claude-crusts（1 命令扫描）
```

### ccusage

```bash
npm i -g ccusage
ccusage          # 今日使用量
ccusage --days 7 # 最近 7 天
```

直接从 `~/.claude/projects/**/*.jsonl` 读取。无 API 密钥，不向外部发送数据。来源：[github.com/ryoppippi/ccusage](https://github.com/ryoppippi/ccusage)。

### claude-code-otel

将 Claude Code 活动导出为 OpenTelemetry spans：

```bash
npm i -g claude-code-otel
claude-code-otel --collector http://localhost:4318
```

Spans 包含工具名称、持续时间、Token（词元）计数。可接入任意 OTEL 兼容后端（Jaeger、Tempo、Datadog）。来源：[github.com/badger-99/claude-code-otel](https://github.com/badger-99/claude-code-otel)。

### ccboard

```bash
cargo install ccboard
ccboard              # 启动 TUI
ccboard --web        # 启动 Web UI（localhost:3000）
```

来源：[github.com/claude-code-ultimate-guide/ccboard](https://github.com/claude-code-ultimate-guide/ccboard)。涵盖文件访问、bash 命令和网络调用的活动标签正在计划中（见 `docs/resource-evaluations/ccboard-activity-module-plan.md`）。

### MLflow Tracing

**适用场景**：已在 MLflow/MLOps 生态系统中的团队，或需要精确 Token（词元）计数 + LLM 质量评估的任何人。对于想要快速成本数字的单独开发者不适合（使用 ccusage 代替）。

**与其他工具的区别**：MLflow 在 API 层拦截，而非事后从 JSONL 获取。它捕获**精确**的 Token（词元）计数（而非基于 Hook 估算的约 15-25% 误差），并支持**LLM 作为评判者**的回归检测——不仅是"发生了什么"，而是"做得好吗？"。

#### 设置：CLI 模式（无需 Python）

适用于交互式 `claude` 会话。Hook 到 `.claude/settings.json`：

```bash
pip install "mlflow[genai]>=3.4"

# 在当前项目目录启用追踪
mlflow autolog claude

# 使用自定义后端（推荐用于持久化）
mlflow autolog claude -u sqlite:///mlflow.db

# 使用命名实验
mlflow autolog claude -n "my-project"

# 检查状态/禁用
mlflow autolog claude --status
mlflow autolog claude --disable
```

启动 UI 查看追踪：

```bash
mlflow server  # → http://localhost:5000
```

**自动捕获的内容**：用户提示词、助手响应、工具调用（名称 + 输入 + 输出）、Token（词元）计数（精确）、每次调用的延迟、会话元数据。

#### 设置：SDK 模式（Python 智能体）

```python
import mlflow
mlflow.anthropic.autolog()         # 一行代码，在任何其他操作之前
mlflow.set_experiment("my-agent")

# 正常使用 ClaudeSDKClient——所有交互都被追踪
# ⚠️ 仅支持 ClaudeSDKClient。直接 API 调用不被追踪。
from anthropic import claude_agent_sdk
async with ClaudeSDKClient(options=AGENT_OPTIONS) as client:
    await client.query(query)
```

需要：`mlflow>=3.5` + `claude-agent-sdk>=0.1.0`。

#### MCP 服务器：双向集成

Claude Code 可以直接查询自己的追踪数据。添加到 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "mlflow-mcp": {
      "command": "uv",
      "args": ["run", "--with", "mlflow[mcp]>=3.5.1", "mlflow", "mcp", "run"],
      "env": { "MLFLOW_TRACKING_URI": "<your-tracking-uri>" }
    }
  }
}
```

配置后，你可以问 Claude Code："找出所有 backend-architect 智能体使用超过 20 次工具调用的会话"——它直接查询 MLflow，无需复制粘贴 ID。

#### LLM 作为评判者：智能体回归检测

这是本节所有其他工具都没有的关键能力。修改智能体指令后，测量质量是否改善或下降：

```python
from mlflow.genai.scorers import scorer, ConversationCompleteness, RelevanceToQuery
from mlflow.entities.model_registry import Feedback

@scorer
def tool_efficiency(trace) -> int:
    """统计工具调用——对于范围明确的任务，越少越好。"""
    return len(trace.search_spans(span_type="TOOL"))

@scorer
def permission_blocks(trace) -> int:
    """检测智能体被权限门阻止的频率。"""
    return sum(
        1 for span in trace.search_spans(span_type="TOOL")
        if span.outputs and "requires approval" in str(span.outputs).lower()
    )

# 对记录的追踪运行评估
traces = mlflow.search_traces(experiment_ids=["<id>"], max_results=50)
results = mlflow.genai.evaluate(
    data=traces,
    scorers=[
        tool_efficiency,
        permission_blocks,
        ConversationCompleteness(),
        RelevanceToQuery(),
    ]
)
```

**内置评估器**：`ConversationCompleteness`、`RelevanceToQuery`、`UserFrustration`、`SafetyScorer`。

**自定义评估器**：完整访问追踪对象（所有 spans、输入、输出、Token（词元）计数）。

#### 局限性

| 局限性 | 详情 |
|------------|--------|
| **CLI 模式受众** | 最适合交互式会话；编程式智能体需要 SDK 模式 |
| **SDK 限制** | 仅支持 `ClaudeSDKClient`——直接 API 调用绕过追踪 |
| **PII 风险** | 追踪捕获完整对话内容。处理敏感数据时存储前需脱敏 |
| **生产后端** | SQLite = 仅开发。生产使用 PostgreSQL/MySQL |
| **OpenTelemetry** | MLflow 3.6+ 可导出到任意 OTEL 兼容后端（Datadog、Grafana 等） |

---

## 代理 Claude Code

常见问题："我可以运行 Proxyman/Charles 来查看 Claude Code 向 Anthropic 发送什么吗？"

**简短回答**：不能直接查看。原因和可行替代方案如下。

### 为什么系统代理不起作用

Claude Code 是 Node.js 进程。默认情况下，Node.js 忽略系统级代理设置（`HTTP_PROXY`、`HTTPS_PROXY`）——它使用自己的 TLS 栈，不读取 macOS/Windows 代理配置。

此外，即使流量流经代理，TLS 证书不匹配也会导致 Claude Code 失败（`CERT_UNTRUSTED`）。

### 选项 1：信任 MITM 证书（Proxyman / Charles）

强制 Node.js 信任你的代理 CA 证书：

```bash
# 导出 Proxyman 的 CA 证书（文件 → 导出 → 根证书）
# 然后让 Node.js 指向它：
export NODE_EXTRA_CA_CERTS="/path/to/proxyman-ca.pem"

# 启动 Claude Code——流量现在将通过 Proxyman 路由
claude
```

Charles 同样适用：`帮助 → SSL 代理 → 导出 Charles 根证书`。

**注意事项**：
- 某些 Claude Code 版本对 `api.anthropic.com` 使用证书固定——这可能仍然失败
- 此方法需要 Proxyman/Charles 实例在配置端口上监听

### 选项 2：用 ANTHROPIC_API_URL 重定向 API 流量

将 Claude Code 指向本地拦截器而非 `api.anthropic.com`：

```bash
export ANTHROPIC_API_URL="http://localhost:8080"
claude
```

在 8080 端口运行任何 HTTP 代理/记录器，将请求转发到 `https://api.anthropic.com`。这完全绕过了 Claude Code → 代理跳转的 TLS。

**用例**：记录请求载荷、注入头部、本地速率限制、重放请求。

### 选项 3：mitmproxy（推荐）

[mitmproxy](https://mitmproxy.org) 是最干净的开源解决方案。它提供带 Web UI 和终端界面的可脚本化 HTTPS 代理。

```bash
# 安装
brew install mitmproxy  # macOS
# 或：pip install mitmproxy

# 在 8080 端口启动透明代理
mitmproxy --listen-port 8080

# 在新终端中，让 Claude Code 指向它
export NODE_EXTRA_CA_CERTS="$(python3 -c 'import mitmproxy.certs; print(mitmproxy.certs.Cert.default_ca_path())')"
export HTTPS_PROXY="http://localhost:8080"
claude
```

`http://localhost:8081` 的 mitmproxy Web UI（`mitmweb`）显示完整的请求/响应体——包括 Claude Code 向 Anthropic 发送的 JSON 载荷。

**你将看到**：系统提示、用户消息、工具定义、工具结果、模型参数。

### 选项 4：最小化 Python 日志代理

零依赖方法：

```python
# proxy.py — 简单的 HTTPS 日志代理
from http.server import HTTPServer, BaseHTTPRequestHandler
import urllib.request, json, sys

TARGET = "https://api.anthropic.com"

class LoggingProxy(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers["Content-Length"])
        body = self.rfile.read(length)
        print(json.dumps(json.loads(body), indent=2))  # 记录请求
        # 转发到 Anthropic...

HTTPServer(("localhost", 8080), LoggingProxy).serve_forever()
```

```bash
python3 proxy.py &
export ANTHROPIC_API_URL="http://localhost:8080"
claude
```

> **隐私说明**：代理流量包含对话上下文中的所有内容——Claude 读取的文件内容、你的代码、任何遇到的 secrets。相应处理代理日志。

---

## 模式与最佳实践

### 1. 每周回顾

设置日历提醒查看每周统计：

```bash
session-stats.sh --range week
```

关注：
- 异常高的 Token（词元）使用天数
- 相同文件上的重复操作（低效信号）
- 项目分布（时间花在哪里）

### 2. 按项目追踪

使用 `CLAUDE_SESSION_ID` 按项目标记会话：

```bash
export CLAUDE_SESSION_ID="project-myapp-$(date +%s)"
claude
```

### 3. 团队聚合

对于团队级追踪，将日志同步到共享存储：

```bash
# 示例：每日同步到 S3
aws s3 sync ~/.claude/logs/ s3://company-claude-logs/$(whoami)/
```

然后聚合：

```bash
# 下载所有团队日志
aws s3 sync s3://company-claude-logs/ /tmp/team-logs/

# 合并和分析
cat /tmp/team-logs/*/activity-$(date +%Y-%m-%d).jsonl | \
  jq -s 'group_by(.project) | map({project: .[0].project, total_tokens: [.[].tokens.total] | add})'
```

### 4. 日志轮转

日志随时间积累。添加到 cron：

```bash
# 清理超过 30 天的日志
find ~/.claude/logs -name "*.jsonl" -mtime +30 -delete
```

---

## 局限性

### 此监控无法做到的事

| 局限性 | 原因 |
|------------|--------|
| **精确 Token（词元）计数** | Claude Code CLI 不暴露 API Token（词元）指标 |
| **TTFT（首 Token（词元）时间）** | Hook 在工具完成后运行，而非流式传输期间 |
| **实时流式指标** | 响应生成期间无 Hook 事件 |
| **实际 API 成本** | Token（词元）估算是启发式，非计费数据 |
| **模型选择** | 日志不捕获每次请求使用的是哪个模型 |
| **上下文窗口使用率** | 对当前上下文百分比无可见性 |

### 准确性说明

- **Token（词元）估算**：与实际计费相差约 15-25%
- **成本估算**：作为方向性参考，而非会计依据
- **会话边界**：会话通过 ID 近似，而非精确的 API 会话

### 可以信任的内容

- **工具使用计数**：每次工具调用的精确次数
- **文件访问模式**：哪些文件被触碰
- **相对比较**：日间/项目间趋势
- **操作时间**：工具使用的时间戳

---

---

## 管理者审计清单

对于需要验证 Claude Code 在团队中使用是否恰当的工程管理者和团队负责人，以下是实用审计查询。

### 每周抽查（5 分钟）

```bash
# 本周是否发生了异常情况？

# 1. 项目范围外访问的文件
find ~/.claude/projects/ -name "*.jsonl" -newer "$(date -d '7 days ago' +%Y-%m-%d 2>/dev/null || date -v-7d +%Y-%m-%d)" 2>/dev/null | \
  xargs jq -r 'select(.type == "assistant") |
    .message.content[]? |
    select(.type == "tool_use" and .name == "Read") |
    .input.file_path' 2>/dev/null | \
  grep -v "^$(pwd)" | sort -u

# 2. 执行的破坏性命令
find ~/.claude/projects/ -name "*.jsonl" -newer "$(date -d '7 days ago' +%Y-%m-%d 2>/dev/null || date -v-7d +%Y-%m-%d)" 2>/dev/null | \
  xargs jq -r 'select(.type == "assistant") |
    .message.content[]? |
    select(.type == "tool_use" and .name == "Bash") |
    .input.command' 2>/dev/null | \
  grep -iE "(drop|delete|truncate|rm -rf|git push --force)"
```

### 合规报告

对于受监管环境，生成 AI 活动摘要供审计员使用：

```bash
#!/bin/bash
# Claude Code 活动的月度合规报告

START_DATE=${1:-$(date -d '30 days ago' +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)}
END_DATE=${2:-$(date +%Y-%m-%d)}
REPORT_FILE="ai-activity-report-${START_DATE}-${END_DATE}.json"

echo "生成合规报告：$START_DATE 至 $END_DATE"

# 统计会话、工具调用和文件访问
jq -s '{
  report_period: {start: "'"$START_DATE"'", end: "'"$END_DATE"'"},
  tool_usage: (group_by(.tool) | map({tool: .[0].tool, count: length})),
  unique_files_accessed: ([.[].file] | unique | length),
  sessions: ([.[].session_id] | unique | length)
}' ~/.claude/logs/activity-*.jsonl 2>/dev/null > "$REPORT_FILE" || \
  echo "未找到活动日志。设置 session-logger.sh hook 以启用。"

echo "报告已保存：$REPORT_FILE"
```

完整治理设置（含自动审计追踪日志记录）见[企业 AI 治理 §6.2](../security/enterprise-governance.md#62-audit-trail-setup)。

---

## 相关资源

- [会话搜索脚本](../../examples/scripts/session-search.sh) — 快速会话搜索与恢复
- [会话记录器 Hook](../../examples/hooks/bash/session-logger.sh)
- [统计分析脚本](../../examples/scripts/session-stats.sh)
- [企业 AI 治理](../security/enterprise-governance.md) — 组织级治理、审计追踪、合规
- [第三方工具](../ecosystem/third-party-tools.md) — 社区 GUI、TUI 和仪表板（ccusage、ccburn、claude-code-viewer）
- [数据隐私指南](../security/data-privacy.md) — 什么数据离开你的机器
- [成本优化](#cost-optimization-tips) — 减少支出的建议
