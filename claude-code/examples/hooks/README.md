> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Claude Code 钩子"
description: "用于自动化、安全拦截和上下文注入的事件驱动脚本"
tags: [hooks, security, template, config]
---

# Claude Code 钩子

钩子是在 Claude Code 事件触发时自动执行的脚本。它们可以实现自动化、拦截危险操作，并丰富上下文信息。

## 可用钩子

| 钩子 | 事件 | 用途 | 平台 |
|------|-------|---------|----------|
| [dangerous-actions-blocker.sh](./bash/dangerous-actions-blocker.sh) | PreToolUse | 拦截危险命令/编辑操作 | Bash |
| [security-check.sh](./bash/security-check.sh) | PreToolUse | 拦截命令中的密钥 | Bash |
| [prompt-injection-detector.sh](./bash/prompt-injection-detector.sh) | PreToolUse | 检测注入攻击尝试（含 ANSI、空字节、嵌套命令） | Bash |
| [unicode-injection-scanner.sh](./bash/unicode-injection-scanner.sh) | PreToolUse | 检测零宽字符、RTL 覆盖、ANSI 转义、空字节 | Bash |
| [repo-integrity-scanner.sh](./bash/repo-integrity-scanner.sh) | PreToolUse | 扫描 README/package.json 中的隐藏注入 | Bash |
| [mcp-config-integrity.sh](./bash/mcp-config-integrity.sh) | SessionStart | 验证 MCP 配置哈希值（CVE-2025-54135/54136） | Bash |
| [claudemd-scanner.sh](./bash/claudemd-scanner.sh) | SessionStart | 检测 CLAUDE.md 注入攻击 | Bash |
| [output-secrets-scanner.sh](./bash/output-secrets-scanner.sh) | PostToolUse | 检测工具输出中的密钥泄漏和环境变量泄漏 | Bash |
| [auto-format.sh](./bash/auto-format.sh) | PostToolUse | 编辑后自动格式化 | Bash |
| [rtk-baseline.sh](./bash/rtk-baseline.sh) | SessionStart | 保存 RTK 基线以追踪会话节省量 | Bash |
| [session-summary.sh](./bash/session-summary.sh) | SessionEnd | 完整会话分析（15 个可配置模块） | Bash |
| [session-summary-config.sh](./bash/session-summary-config.sh) | CLI 工具 | 配置 session-summary 模块、顺序及预览 | Bash |
| [learning-capture.sh](./bash/learning-capture.sh) | Stop | 提示每日学习记录 | Bash |
| [sandbox-validation.sh](./bash/sandbox-validation.sh) | PreToolUse | 验证沙箱隔离状态 | Bash |
| [file-guard.sh](./bash/file-guard.sh) | PreToolUse | 保护敏感文件不被修改 | Bash |
| [permission-request.sh](./bash/permission-request.sh) | PreToolUse | 对高风险操作进行显式授权流程 | Bash |
| [rtk-auto-wrapper.sh](./bash/rtk-auto-wrapper.sh) | PreToolUse | 自动为命令包装 RTK 以节省 Token | Bash |
| [setup-init.sh](./bash/setup-init.sh) | SessionStart | 初始化会话环境 | Bash |
| [auto-checkpoint.sh](./bash/auto-checkpoint.sh) | PostToolUse | 定期自动保存检查点 | Bash |
| [typecheck-on-save.sh](./bash/typecheck-on-save.sh) | PostToolUse | 保存时运行 TypeScript 检查 | Bash |
| [test-on-change.sh](./bash/test-on-change.sh) | PostToolUse | 文件变更时运行测试 | Bash |
| [output-validator.sh](./bash/output-validator.sh) | PostToolUse | 启发式输出验证 | Bash |
| [session-logger.sh](./bash/session-logger.sh) | PostToolUse | 记录操作日志用于监控 | Bash |
| [privacy-warning.sh](./bash/privacy-warning.sh) | PostToolUse | 警告潜在的隐私泄漏 | Bash |
| [tts-selective.sh](./bash/tts-selective.sh) | PostToolUse | 对选定输出进行文字转语音 | Bash |
| [subagent-stop.sh](./bash/subagent-stop.sh) | Stop | 清理子代理资源 | Bash |
| [pre-commit-secrets.sh](./bash/pre-commit-secrets.sh) | Git hook | 阻止密钥进入提交 | Bash |
| [pre-commit-evaluator.sh](./bash/pre-commit-evaluator.sh) | Git hook | LLM 作为评审员的提交前验证 | Bash |
| [notification.sh](./bash/notification.sh) | Notification | 基于上下文的 macOS 音效提醒 | Bash（macOS） |
| [auto-rename-session.sh](./bash/auto-rename-session.sh) | SessionEnd | 通过 Haiku 模型 AI 自动生成会话标题 | Bash |
| [security-gate.sh](./bash/security-gate.sh) | PreToolUse | 在写入源文件前检测易受攻击的代码模式 | Bash |
| [velocity-governor.sh](./bash/velocity-governor.sh) | PreToolUse | 限制工具调用频率以避免 API 限流 | Bash |
| [security-check.ps1](./powershell/security-check.ps1) | PreToolUse | 拦截命令中的密钥 | PowerShell |
| [auto-format.ps1](./powershell/auto-format.ps1) | PostToolUse | 编辑后自动格式化 | PowerShell |

## 钩子事件

| 事件 | 触发时机 | 典型用途 |
|-------|------|-------------------|
| `SessionStart` | 会话开始或恢复时 | 初始化、环境设置、配置扫描 |
| `UserPromptSubmit` | 用户发送消息时 | 上下文注入、预处理 |
| `PreToolUse` | 工具执行前 | 验证、拦截危险操作 |
| `PermissionRequest` | 权限对话框出现时 | 自定义审批逻辑 |
| `PostToolUse` | 工具成功执行后 | 格式化、日志记录、清理 |
| `PostToolUseFailure` | 工具执行失败后 | 错误日志、恢复操作 |
| `Notification` | Claude 发送通知时 | 音效提醒、外部通知 |
| `SubagentStart` | 子代理生成时 | 子代理初始化 |
| `SubagentStop` | 子代理结束时 | 子代理清理 |
| `Stop` | Claude 完成响应时 | 响应后操作、状态保存 |
| `TeammateIdle` | 代理队友空闲时 | 团队协调 |
| `TaskCompleted` | 任务标记完成时 | 工作流触发 |
| `ConfigChange` | 会话期间配置文件变更时 | 企业审计、阻止未授权变更 |
| `WorktreeCreate` | 代理工作树创建时 | 创建数据库分支、安装依赖 |
| `WorktreeRemove` | 代理工作树销毁时 | 清理数据库分支、临时凭证 |
| `PreCompact` | 上下文压缩前 | 压缩前保存状态 |
| `SessionEnd` | 会话终止时 | 清理、会话摘要 |

## 高级防护（v3.3.0 新增）

受生产 LLM 系统启发的高级防护模式。

### prompt-injection-detector.sh

**事件**：`PreToolUse`

在注入攻击到达 Claude 之前进行检测和拦截：

**检测模式**：
- 角色覆盖："ignore previous instructions"、"you are now"、"pretend to be"
- 越狱尝试："DAN mode"、"developer mode"、"no restrictions"
- 分隔符注入：`</system>`、`[INST]`、`<<SYS>>`
- 权限伪造："anthropic employee"、"authorized to bypass"
- Base64 编码载荷（解码后扫描）
- 上下文操控：对之前消息的虚假声称

**配置**：
```json
{
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/prompt-injection-detector.sh",
        "timeout": 5000
      }]
    }]
  }
}
```

### output-validator.sh

**事件**：`PostToolUse`

对 Claude 输出进行启发式验证（无需 LLM 调用，纯 Bash 实现）：

**验证检查**：
- 占位符路径：`/path/to/`、`/your/project/`
- 占位符内容：`TODO:`、`your-api-key`、`example.com`
- 输出中的潜在密钥（正则匹配）
- 不确定性指示词（多次出现"I'm not sure"、"probably"）
- 不完整实现：`NotImplementedError`、`throw new Error`
- 未经验证的引用声明

**行为**：通过 `systemMessage` 发出警告，不进行拦截。如需更深层的验证，请使用 `output-evaluator` 代理。

### session-logger.sh

**事件**：`PostToolUse`

将所有 Claude 操作记录到 JSONL 文件，用于监控和费用追踪：

**日志位置**：`~/.claude/logs/activity-YYYY-MM-DD.jsonl`

**记录内容**：
- 时间戳、会话 ID、工具名称
- 文件路径和命令（截断处理）
- 项目名称
- Token 估算（输入/输出）

**分析**：使用 `session-stats.sh` 脚本分析日志。

**环境变量**：
| 变量 | 默认值 | 说明 |
|----------|---------|-------------|
| `CLAUDE_LOG_DIR` | `~/.claude/logs` | 日志目录 |
| `CLAUDE_LOG_TOKENS` | `true` | 启用 Token 估算 |
| `CLAUDE_SESSION_ID` | 自动 | 自定义会话 ID |

详见[可观测性指南](../../guide/ops/observability.md)。

### pre-commit-evaluator.sh

**类型**：Git 提交前钩子（非 Claude 钩子）

每次提交前进行 LLM 作为评审员的评估。**需手动开启**，因为会产生 API 费用。

**安装**：
```bash
cp pre-commit-evaluator.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
export CLAUDE_PRECOMMIT_EVAL=1  # Enable evaluation
```

**费用**：每次提交约 $0.01-0.05（Haiku 模型）

**跳过方式**：`git commit --no-verify` 或 `CLAUDE_SKIP_EVAL=1 git commit`

---

## 安全类钩子

### dangerous-actions-blocker.sh

**事件**：`PreToolUse`（Bash、Edit、Write）

针对危险操作的全面防护：

**Bash - 被拦截的命令**：
- 系统破坏：`rm -rf /`、`rm -rf ~`、`sudo rm`
- 磁盘操作：`dd if=`、`mkfs`、`> /dev/sda`
- Fork 炸弹：`:(){:|:&};:`
- 数据库删除：`DROP DATABASE`、`DROP TABLE`
- 强制推送：`git push --force main/master`
- 包发布：`npm publish`、`pnpm publish`
- 密钥模式：`password=`、`api_key=`、`token=`

**Edit/Write - 受保护的文件**：
- 环境变量：`.env`、`.env.local`、`.env.production`
- 凭证：`credentials.json`、`serviceAccountKey.json`
- SSH 密钥：`id_rsa`、`id_ed25519`、`id_ecdsa`
- 配置：`.npmrc`、`.pypirc`、`secrets.yml`

**Edit/Write - 允许的路径**：
- `$CLAUDE_PROJECT_DIR`（当前项目）
- `~/.claude/`（Claude 配置）
- `/tmp/`（临时文件）
- 通过 `$ALLOWED_PATHS` 环境变量添加的额外路径

**退出码**：
```bash
exit 0  # Allow operation
exit 2  # Block (stderr message shown to Claude)
```

**配置**：
```bash
# Add custom allowed paths (colon-separated)
export ALLOWED_PATHS="/custom/path:/another/path"
```

### security-check.sh

**事件**：`PreToolUse`（Bash）

专注于检测命令中的密钥：
- 密码模式
- API 密钥（常见格式如 `sk-xxx`、`pk-xxx`）
- AWS 凭证
- 私钥
- 硬编码 Token

### claudemd-scanner.sh

**事件**：`SessionStart`

在会话开始时扫描 CLAUDE.md 文件，检测潜在的提示注入攻击：

**检测模式**：
- "ignore previous instructions" 变体
- Shell 注入：`curl | bash`、`wget | sh`、`eval(`
- Base64 编码内容（潜在混淆）
- HTML 注释中的隐藏指令
- 异常长行（>500 字符）
- 敏感关键词附近的非 ASCII 字符（同形字攻击）

**扫描的文件**：
- `CLAUDE.md`（项目根目录）
- `.claude/CLAUDE.md`（本地覆盖）
- `.claude/` 目录中的所有 `.md` 文件

**为何重要**：克隆陌生仓库时，恶意的 CLAUDE.md 可能注入指令从而危害您的系统。此钩子会在 Claude 处理潜在危险指令前发出警告。

**配置**：
```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/claudemd-scanner.sh",
        "timeout": 5000
      }]
    }]
  }
}
```

### output-secrets-scanner.sh

**事件**：`PostToolUse`

补充 `security-check.sh` 的功能，扫描工具**输出**（而非输入）中的密钥泄漏。

**检测模式**：
- API 密钥：OpenAI、Anthropic、AWS、GCP、Azure、Stripe、Twilio、SendGrid
- Token：GitHub、GitLab、NPM、PyPI、JWT
- 私钥：RSA、EC、DSA、OpenSSH、PGP
- 含嵌入密码的数据库 URL
- 通用的 `api_key=`、`secret=`、`password=` 模式

**为何重要**：Claude 可能读取 `.env` 文件并将凭证包含在响应或提交中。此钩子在密钥泄漏前将其拦截。

**配置**：
```json
{
  "hooks": {
    "PostToolUse": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/output-secrets-scanner.sh",
        "timeout": 5000
      }]
    }]
  }
}
```

## 效率类钩子

### rtk-baseline.sh

**事件**：`SessionStart`

在会话开始时捕获 RTK 累计统计基线，用于增量追踪。与 `session-summary.sh` 配合使用，后者会在会话结束时计算每次会话的 RTK 节省量。

**配置**：
```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/rtk-baseline.sh",
        "timeout": 5000
      }]
    }]
  }
}
```

### session-summary.sh（v3）

**事件**：`SessionEnd`

完整会话分析，包含 15 个可配置模块、CLI 配置工具及 JSONL 日志。

**插件安装（推荐）**：
```bash
claude plugin marketplace add claude-code-ultimate-guide/claude-code-plugins
claude plugin install session-summary@florian-claude-tools
```
钩子自动配置，无需手动设置。详见[插件仓库](https://github.com/claude-code-ultimate-guide/claude-code-plugins)。

**模块**（均可通过环境变量或配置文件调整）：

| 模块 | 默认状态 | 说明 |
|---------|---------|-------------|
| `meta` | 始终开启 | 会话 ID、名称、分支 |
| `duration` | 始终开启 | 总时长、活跃时长、轮次、退出原因 |
| `tools` | 始终开启 | 工具调用明细（成功/失败） |
| `errors` | 开启 | 按工具分组的错误详情 |
| `files` | 开启 | 读取/编辑/创建的文件及编辑最多的文件 |
| `features` | 开启 | MCP 服务器、代理、技能、团队、计划模式 |
| `git` | 开启 | Git 差异摘要（增减行数、变更文件数） |
| `loc` | 开启 | 通过 Edit/Write 写入的代码行数 |
| `models` | 始终开启 | 模型使用情况（请求数、Token 数） |
| `cache` | 始终开启 | 缓存命中率 |
| `cost` | 始终开启 | 估算费用（ccusage 或定价表） |
| `rtk` | 开启 | RTK Token 节省量（与会话开始的增量） |
| `ratio` | 开启 | 对话比例（交互轮次/自动轮次） |
| `thinking` | 关闭 | 思考块数量 |
| `context` | 关闭 | 上下文窗口估算（峰值百分比） |

**日志文件**：`~/.claude/logs/session-summaries.jsonl`（包含所有指标的结构化 JSONL）

**配置优先级**：`环境变量（SESSION_SUMMARY_*）` > `配置文件（~/.config/session-summary/config.sh）` > `默认值`

**环境变量**：
| 变量 | 默认值 | 说明 |
|----------|---------|-------------|
| `NO_COLOR` | - | 禁用 ANSI 颜色 |
| `SESSION_SUMMARY_SKIP` | `0` | 设为 `1` 禁用摘要 |
| `SESSION_SUMMARY_LOG` | `~/.claude/logs` | 覆盖日志目录 |
| `SESSION_SUMMARY_FILES` | `1` | 文件模块开关 |
| `SESSION_SUMMARY_RTK` | `auto` | `auto` / `1` / `0` |
| `SESSION_SUMMARY_GIT` | `1` | Git 差异开关 |
| `SESSION_SUMMARY_ERRORS` | `1` | 错误详情开关 |
| `SESSION_SUMMARY_LOC` | `1` | 代码行数开关 |
| `SESSION_SUMMARY_RATIO` | `1` | 对话比例开关 |
| `SESSION_SUMMARY_FEATURES` | `1` | 功能使用开关 |
| `SESSION_SUMMARY_THINKING` | `0` | 思考块开关 |
| `SESSION_SUMMARY_CONTEXT` | `0` | 上下文估算开关 |
| `SESSION_SUMMARY_SECTIONS` | （全部） | 以逗号分隔的模块顺序 |

**CLI 配置工具**（`session-summary-config.sh`）：
```bash
session-summary-config show              # Current config with section status
session-summary-config set git=0         # Disable a section
session-summary-config set thinking=1    # Enable a section
session-summary-config reset             # Reset to defaults
session-summary-config sections          # Show section order
session-summary-config sections "meta,duration,tools,cost"  # Minimal output
session-summary-config preview           # Demo output with current config
session-summary-config install           # Install hooks in settings.json
session-summary-config log 5             # Last 5 session summaries
```

**依赖**：
- `jq`（必需，用于 JSON 解析）
- `ccusage`（可选，用于精确费用计算）
- `rtk`（可选，用于 Token 节省追踪）

**截图**（真实会话输出）：

![Session Summary v3](../../docs/images/session-summary-v3.png)

**示例输出**（所有模块开启）：
```
═══ Session Summary ═══════════════════
ID:       abc-123-def-456...
Name:     Fix session summary hook
Branch:   main
Duration: Wall 5m 28s | Active 1m 33s | 12 turns | Exit: user

Tool Calls: 29 (OK 27 / ERR 2)
  Edit: 13  Bash: 8  Read: 6  Grep: 1  Glob: 1

Errors: 2
  Bash: "command not found: rtk" (x1)
  Edit: "old_string not unique" (x1)

Files: 3 read · 2 edited · 1 created
  session-summary.sh (8 edits), settings.json (3 edits)

Features: MCP (perplexity x4, chrome x12) · Agents (Explore x3, Plan x1) · Skills (commit)

Git: +142 -37 lines · 4 files changed
Code: +87 -12 net (via Edit/Write)

Model Usage         Reqs    Input    Output
claude-opus-4-6       59      93K      628
Cache: 85% hit rate (3.9M read / 322K created)
Est. Cost: $0.045

RTK Savings: 24 cmds · ~12.4K tokens saved (73%)
Turns: 12 (8 interactive · 4 auto) · Avg 6.7s/turn
═══════════════════════════════════════
```

**配置**：
```json
{
  "hooks": {
    "SessionEnd": [{
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/session-summary.sh"
      }]
    }]
  }
}
```

**快速安装**：插件系统（见上文）或手动安装：`session-summary-config.sh install`（复制钩子并更新 settings.json）

**与 tweakcc 的 `/cost` 补丁对比**：

[tweakcc](https://github.com/Piebald-AI/tweakcc)（1K+ 星）通过修补 Claude Code 的 `cli.js` 为 Pro/Max 订阅者重新启用 `/cost` 功能。两者方式不同，各有利弊：

| | tweakcc `/cost` | session-summary.sh |
|---|---|---|
| 方式 | 修补 Claude Code 二进制文件 | 官方钩子 API（无需修改） |
| 是否随 CC 更新存活 | 否（每次更新后需重新应用） | 是 |
| 触发方式 | 手动（`/cost` 命令） | 会话退出时自动触发 |
| 指标 | 费用、时长、Token、代码行数 | 15 个模块（费用、Token、工具、错误、文件、功能、Git 差异、代码行数、缓存、RTK、比例……） |
| 历史记录 | 无 | 含所有指标的 JSONL 日志 |
| 依赖 | Node.js | jq（Bash 原生） |

tweakcc 是一个更广泛的工具（主题、提示词、工具集），`/cost` 补丁只是其众多功能之一。本钩子专注于会话分析，提供更深层的指标，且不修改 Claude Code 内部文件。

### auto-format.sh / auto-format.ps1

**事件**：`PostToolUse`（Edit、Write）

编辑后自动格式化文件：

| 扩展名 | 格式化工具 |
|-----------|-----------|
| `.ts`、`.tsx`、`.js`、`.jsx` | Prettier |
| `.json`、`.css`、`.scss`、`.md` | Prettier |
| `.prisma` | `prisma format` |
| `.py` | Black / autopep8 |
| `.go` | `go fmt` |

**静默运行**：无输出，忽略失败以避免阻塞 Claude。

**依赖**：在项目中安装格式化工具：
```bash
# Node.js projects
npm install -D prettier

# Python projects
pip install black

# Go projects (built-in)
go fmt
```

### notification.sh

**事件**：`Notification`（仅限 macOS）

根据通知内容触发对应音效提醒：

| 上下文 | 音效 | 触发词 |
|---------|-------|--------------|
| 成功 | Hero.aiff | "completed"、"done"、"success" |
| 错误 | Basso.aiff | "error"、"failed"、"problem" |
| 等待 | Submarine.aiff | "waiting"、"permission"、"input" |
| 警告 | Sosumi.aiff | "warning"、"attention"、"alert" |
| 默认 | Ping.aiff | 其他通知 |

**特性**：
- 非阻塞（后台播放）
- 原生 macOS 通知
- 通过关键词自动识别上下文
- 多语言支持（英语/法语）

**依赖**：macOS 系统内置的 `afplay` 和 `osascript`

## 配置

钩子在 `.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/dangerous-actions-blocker.sh",
          "timeout": 5000
        }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/auto-format.sh",
          "timeout": 10000
        }]
      }
    ],
    "Notification": [
      {
        "hooks": [{
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/notification.sh",
          "timeout": 5000
        }]
      }
    ]
  }
}
```

**Matcher 模式**：
- `".*"` - 匹配所有工具
- `"Bash"` - 仅匹配 Bash 工具
- `"Edit|Write"` - 匹配 Edit 或 Write 工具
- `"Bash|Edit|Write"` - 匹配多个工具

## 创建自定义钩子

### 基础模板

```bash
#!/bin/bash
# Hook: [EventType] - Description
# Exit 0 = success/allow, Exit 2 = block (PreToolUse only)

set -e

# Read JSON from stdin
INPUT=$(cat)

# Extract data
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name // empty')
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input // empty')

# Your logic here...

# Return optional JSON
cat << EOF
{
  "systemMessage": "Message displayed to Claude",
  "hookSpecificOutput": {
    "additionalContext": "Extra context added"
  }
}
EOF

exit 0
```

### 环境变量

钩子脚本中可用的变量：

| 变量 | 说明 |
|----------|-------------|
| `CLAUDE_PROJECT_DIR` | 当前项目路径 |
| `CLAUDE_FILE_PATHS` | 通过 `-f` 标志传入的文件 |
| `CLAUDE_TOOL_INPUT` | JSON 格式的工具输入 |
| `HOME` | 用户主目录 |

### 最佳实践

1. **控制超时**：最长 5-10 秒，避免阻塞 Claude
2. **优雅降级**：非关键操作使用 `|| true`
3. **最小化日志**：除结构化 JSON 外避免输出到 stdout
4. **依赖 jq**：使用 `jq` 解析 JSON 以确保可靠性
5. **充分测试**：部署前用各种输入进行测试
6. **文档化行为**：用清晰注释说明钩子的作用
7. **处理错误**：提供清晰的错误信息便于调试

### 示例：Git 上下文注入

创建 `git-context.sh`（UserPromptSubmit 事件）：

```bash
#!/bin/bash
# Hook: UserPromptSubmit - Add Git context to prompts

set -e

INPUT=$(cat)

# Get Git information
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "unknown")
LAST_COMMIT=$(git log -1 --oneline 2>/dev/null || echo "none")
UNCOMMITTED=$(git status --short 2>/dev/null | wc -l | tr -d ' ')
STAGED=$(git diff --cached --name-only 2>/dev/null | wc -l | tr -d ' ')

# Return enriched context
cat << EOF
{
  "systemMessage": "[Git] Branch: $BRANCH | Last: $LAST_COMMIT | Uncommitted: $UNCOMMITTED files | Staged: $STAGED files"
}
EOF

exit 0
```

在 settings 中注册：
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [{
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/git-context.sh",
          "timeout": 3000
        }]
      }
    ]
  }
}
```

## 安装

### 项目级别（与团队共享）

1. 创建钩子目录：
```bash
mkdir -p .claude/hooks
```

2. 从示例中复制钩子：
```bash
cp /path/to/examples/hooks/bash/dangerous-actions-blocker.sh .claude/hooks/
chmod +x .claude/hooks/*.sh
```

3. 在 `.claude/settings.json` 中配置（参见上方配置章节）

4. 提交到仓库：
```bash
git add .claude/hooks/ .claude/settings.json
git commit -m "Add Claude Code hooks"
```

### 个人/全局级别（仅限本机）

1. 创建全局钩子目录：
```bash
mkdir -p ~/.claude/hooks
```

2. 复制钩子：
```bash
cp /path/to/examples/hooks/bash/notification.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/*.sh
```

3. 在 `~/.claude/settings.json` 中配置

**优先级**：项目钩子优先于全局钩子。

## 平台说明

### macOS / Linux（Bash）

- 使用 `.sh` 扩展名
- 需要 `chmod +x` 授予执行权限
- 路径分隔符：`/`
- 主目录：`~` 或 `$HOME`

### Windows（PowerShell）

- 使用 `.ps1` 扩展名
- 可能需要设置执行策略：`Set-ExecutionPolicy RemoteSigned`
- 路径分隔符：`\`
- 主目录：`$env:USERPROFILE`

## 故障排查

### 钩子未执行

**原因**：未在 settings.json 中注册，或路径错误

**解决**：验证配置，使用绝对路径或 `$CLAUDE_PROJECT_DIR`

### 权限被拒绝

**原因**：钩子没有执行权限

**解决**：
```bash
chmod +x .claude/hooks/*.sh
```

### 钩子拦截所有操作

**原因**：无条件使用了退出码 2

**解决**：检查逻辑，确保默认情况下使用 `exit 0`

### 超时错误

**原因**：钩子执行时间超过超时限制

**解决**：优化钩子性能，或在 settings 中增大超时值

### jq 未找到

**原因**：`jq` 未安装

**解决**：安装 jq：
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# Windows
choco install jq
```

## 高级示例

### 操作日志记录器

将所有 Claude 操作记录到 JSONL 文件：

```bash
#!/bin/bash
# PostToolUse - Log all activities

set -e

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')
LOG_FILE="$HOME/.claude/logs/activity-$(date +%Y-%m-%d).jsonl"

mkdir -p "$(dirname "$LOG_FILE")"

# Create log entry
cat << EOF >> "$LOG_FILE"
{"timestamp":"$(date -u +%Y-%m-%dT%H:%M:%SZ)","tool":"$TOOL","session":"$CLAUDE_SESSION_ID"}
EOF

exit 0
```

### 数据库迁移检测器

在创建迁移文件时发出提醒：

```bash
#!/bin/bash
# PostToolUse - Detect database migrations

set -e

INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')

if [[ "$TOOL" == "Write" ]]; then
    FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path')

    if [[ "$FILE" == *"/migrations/"* ]]; then
        cat << EOF
{
  "systemMessage": "⚠️ Database migration created: $FILE\n\nReminder:\n1. Review migration carefully\n2. Test on dev database first\n3. Run 'prisma migrate deploy' after merge"
}
EOF
    fi
fi

exit 0
```

## 安全注意事项

1. **不要在钩子中存储密钥**：使用环境变量
2. **验证输入**：始终对 stdin 数据进行净化处理
3. **限制钩子范围**：使用精确的 matcher，不要用 `".*"`
4. **记录拦截操作**：钩子拦截操作时记录日志
5. **隔离测试**：先在 Claude 外部测试钩子
6. **版本控制**：将钩子提交到仓库以便团队共享

## 相关资源

- [主指南 - 第 7 章：钩子](../../guide/ultimate-guide.md#7-hooks)
- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [钩子事件参考](../../guide/ultimate-guide.md#71-hook-events)

---

*详细说明和高级模式请参见[主指南](../../guide/ultimate-guide.md)。*
