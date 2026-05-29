> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "实用脚本"
description: "Claude Code 高级用户实用脚本：审计、健康检查与会话管理"
tags: [template, debugging, security, workflows]
---

# Scripts

Claude Code 高级用户实用脚本。

## 概览

| 脚本 | 描述 |
|--------|-------------|
| `pptx-to-pdf.sh` | 在 macOS 上通过 Keynote 批量将 PPTX 转换为 PDF（零依赖） |
| `audit-scan.sh` | Claude Code 配置的安全与质量审计 |
| `check-claude.sh/.ps1` | Claude Code 安装健康检查 |
| `clean-reinstall-claude.sh/.ps1` | 彻底重装 Claude Code |
| `fresh-context-loop.sh` | 以全新上下文循环运行 Claude Code |
| `session-search.sh` | 跨 Claude Code 会话历史搜索 |
| `cc-sessions.py` | 带增量索引的高级会话搜索（Python） |
| `session-stats.sh` | Claude Code 会话统计信息 |
| `bridge.py` | 桥接：Claude Code → doobidoo → LM Studio |
| `bridge-plan-schema.json` | 桥接计划 v1 格式的 JSON Schema |
| `migrate-arguments-syntax.sh` | 将 v1 → v2 斜杠命令参数语法迁移（bash） |
| `migrate-arguments-syntax.ps1` | 将 v1 → v2 斜杠命令参数语法迁移（PowerShell） |
| `rtk-benchmark.sh` | 对比 RTK 节省的 token 数量与原始命令 |
| `sync-claude-config.sh` | 在多台机器间同步 Claude 配置文件 |
| `sonnetplan.sh` | 用 Sonnet 替代 Opus 运行 Claude（成本优化别名） |
| `test-prompt-caching.ts` | 验证 Anthropic 提示词缓存是否生效（无依赖，仅 fetch） |
| `smart-suggest-roi.py` | 分析 smart-suggest 钩子建议的采纳率与会话活动对比 |

---

## Bridge 脚本（Claude Code → LM Studio）

**用途**：通过 LM Studio 在本地执行 Claude Code 计划，节省费用。

### 架构

```
┌──────────────┐     store_memory      ┌─────────────────┐
│ Claude Code  │ ─────────────────────►│    doobidoo     │
│   (Opus)     │   tag: "plan"         │   SQLite + Vec  │
│   规划器     │   status: "pending"   │ ~/.mcp-memory-  │
└──────────────┘                       │  service/       │
                                       └────────┬────────┘
                                                │
                                                │ 直接读取 SQLite
                                                ▼
                                       ┌─────────────────┐
                                       │   bridge.py     │
                                       │                 │
                                       │ • PlanReader    │
                                       │ • StepExecutor  │
                                       │ • Validator     │
                                       └────────┬────────┘
                                                │
                                                │ HTTP POST
                                                │ /v1/chat/completions
                                                ▼
                                       ┌─────────────────┐
                                       │    LM Studio    │
                                       │  localhost:1234 │
                                       └─────────────────┘
```

### 依赖要求

```bash
pip install httpx
```

- **doobidoo MCP 服务器**，使用 SQLite 后端（`~/.mcp-memory-service/`）
- **LM Studio** 运行在 `localhost:1234`，并已加载模型

### 使用方法

```bash
# 检查 LM Studio 是否运行
python bridge.py --health

# 列出待执行的计划
python bridge.py --list

# 执行所有待执行计划
python bridge.py

# 执行指定计划
python bridge.py --plan plan_auth_refactor

# 详细模式
python bridge.py -v
```

### 工作流

#### 1. Claude Code 创建计划

在 Claude Code（Opus）中，通过 doobidoo 存储计划：

```
store_memory("""
{
  "$schema": "bridge-plan-v1",
  "id": "plan_auth_refactor",
  "status": "pending",
  "context": {
    "project": "/path/to/project",
    "objective": "将认证重构为使用 JWT",
    "files_context": {
      "src/auth.py": "LOAD",
      "src/config.py": "REFERENCE"
    }
  },
  "steps": [
    {
      "id": 1,
      "type": "analysis",
      "description": "分析当前认证实现",
      "prompt": "分析认证代码，识别 JWT 迁移点。",
      "validation": {"type": "non_empty"}
    },
    {
      "id": 2,
      "type": "code_generation",
      "description": "生成 JWT 中间件",
      "prompt": "根据分析结果生成 JWT 认证中间件。",
      "depends_on": [1],
      "validation": {"type": "syntax_check"},
      "file_output": "src/jwt_auth.py"
    }
  ]
}
""", tags=["plan"])
```

#### 2. 通过 bridge 执行

```bash
python bridge.py
# 从 doobidoo SQLite 读取计划
# 通过 LM Studio 执行每个步骤
# 将结果存回 doobidoo
```

#### 3. 在 Claude Code 中获取结果

```
search_by_tag(["result", "plan_auth_refactor"])
# 返回所有执行结果
```

### 计划 Schema

完整 JSON Schema 请参见 `bridge-plan-schema.json`。

| 字段 | 是否必填 | 描述 |
|-------|----------|-------------|
| `$schema` | 是 | 必须为 `"bridge-plan-v1"` |
| `id` | 是 | 唯一计划 ID（如 `plan_auth_refactor`） |
| `status` | 是 | `pending`、`in_progress`、`completed`、`failed` |
| `context.objective` | 是 | 高层目标描述 |
| `context.project` | 否 | 项目根目录的绝对路径 |
| `context.files_context` | 否 | 要注入（`LOAD`）或引用的文件 |
| `steps` | 是 | 执行步骤数组 |

### 步骤类型

| 类型 | 适用场景 |
|------|----------|
| `analysis` | 分析代码、识别模式、规划变更 |
| `code_generation` | 从零生成新代码 |
| `code_modification` | 修改现有代码 |
| `decision` | 做出架构或设计决策 |

### 验证类型

| 类型 | 描述 |
|------|-------------|
| `non_empty` | 输出不为空（默认） |
| `json` | 有效的 JSON 输出 |
| `syntax_check` | 有效的 Python 语法 |
| `contains_keys` | JSON 包含指定键 |

### 失败处理

| on_failure | 行为 |
|------------|----------|
| `retry_with_context` | 携带错误反馈重试（默认） |
| `skip` | 跳过该步骤，继续执行 |
| `halt` | 终止整个计划 |

### 费用节省

- **规划**（Opus）：每个复杂计划约 $0.50-2.00
- **执行**（LM Studio）：免费（本地）
- **投资回报**：实现任务的费用降低 80-90%

### 局限性

| 局限性 | 缓解措施 |
|------------|------------|
| 本地模型质量参差不齐 | 严格验证 + 重试 |
| LM Studio 不支持 MCP 工具 | 在上下文中注入文件内容 |
| 上下文窗口有限 | 截断旧结果 |
| 不支持流式输出 | 每步 120 秒超时 |

---

## 审计扫描

对 Claude Code 配置进行安全与质量审计。

```bash
./audit-scan.sh
```

检查内容：
- CLAUDE.md 文件中的敏感数据
- 权限配置
- MCP 服务器安全性
- 钩子脚本安全性

---

## 健康检查

快速验证 Claude Code 安装状态。

```bash
# macOS/Linux
./check-claude.sh

# Windows
./check-claude.ps1
```

---

## 彻底重装

完整重装并保留配置。

```bash
# macOS/Linux
./clean-reinstall-claude.sh

# Windows
./clean-reinstall-claude.ps1
```

---

## 全新上下文循环

以全新上下文运行 Claude Code，适合长时间运行的任务。

```bash
./fresh-context-loop.sh --iterations 5 --project /path/to/project
```

---

## 会话搜索

跨所有 Claude Code 会话历史进行搜索。

```bash
./session-search.sh "authentication"
```

---

## 会话管理器（高级）

高级 CLI，支持会话搜索、浏览、恢复与模式发现，具备增量索引能力。

**与 session-search.sh 的对比**：搜索更快（约 200ms vs 约 400ms）、支持部分 ID 恢复、分支过滤、worktree 支持、增量 JSONL 索引，以及用于自动配置优化的 `discover` 子命令。

**GitHub**：[claude-code-ultimate-guide/cc-sessions](https://github.com/claude-code-ultimate-guide/cc-sessions)

```bash
# 在当前项目中搜索
cc-sessions search "notion"

# 搜索所有项目
cc-sessions --all search "stripe"

# 按日期和分支过滤
cc-sessions search "auth" --since 7d --branch develop

# 最近的会话
cc-sessions recent 10

# 通过部分 ID 恢复
cc-sessions resume 8d472d

# JSON 输出（用于脚本）
cc-sessions --json search "prisma" | jq -r '.[].id'

# 发现高频模式（n-gram，本地，免费）
cc-sessions --all discover

# 通过 claude --print 进行语义分析
cc-sessions --all discover --llm

# JSON 输出（用于脚本）
cc-sessions --all discover --json | jq '.[] | select(.category == "skill")'
```

**从 GitHub 安装**：
```bash
curl -sL https://raw.githubusercontent.com/claude-code-ultimate-guide/cc-sessions/main/cc-sessions \
  -o ~/.local/bin/cc-sessions && chmod +x ~/.local/bin/cc-sessions
```

**或本地复制**：`cp cc-sessions.py ~/bin/cc-sessions && chmod +x ~/bin/cc-sessions`

> [GitHub 仓库](https://github.com/claude-code-ultimate-guide/cc-sessions) · [Gist](https://gist.github.com/claude-code-ultimate-guide/992d4d1107592d9e98ca9d89838871c6)

---

## 会话统计

获取 Claude Code 使用情况的统计信息。

```bash
./session-stats.sh
```

---

## PPTX 转 PDF（macOS）

使用 Keynote 批量将 PPTX 演示文稿转换为 PDF。无需 LibreOffice，无需 Python——仅需 macOS + Keynote。

```bash
# 转换文件夹中所有 PPTX（递归）
./pptx-to-pdf.sh ~/Downloads/Prose

# 转换当前目录
./pptx-to-pdf.sh
```

**要求**：macOS + 已安装 Keynote。

**行为说明**：
- 递归查找子目录中所有 `.pptx` 文件
- 幂等操作：已存在对应 `.pdf` 的文件会跳过
- 输出：PDF 与 PPTX 同目录、同名生成
- 结束时打印汇总信息

**重要细节**：脚本通过 shell 的 `open -a "Keynote"` 打开文件，而非使用 AppleScript 自身的 `open` 命令。通过 AppleScript 打开 PPTX 时，Keynote 有时不会将文档注册到内部列表，导致 `document 1` 触发 -1719 错误。使用 shell open + 8 秒等待的方式可稳定解决此问题。
