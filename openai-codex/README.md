# OpenAI Codex 完整指南

> 基于官方文档整理，适合零基础新手入门

---

## 目录

1. [Codex 是什么](#1-codex-是什么)
2. [快速上手](#2-快速上手)
3. [四种使用方式](#3-四种使用方式)
   - [Desktop App（桌面应用）](#31-desktop-app桌面应用)
   - [IDE 扩展](#32-ide-扩展)
   - [CLI（命令行）](#33-cli命令行)
   - [Cloud 云端版](#34-cloud-云端版)
4. [CLI 核心功能详解](#4-cli-核心功能详解)
5. [Slash 命令速查](#5-slash-命令速查)
6. [Skills（技能扩展）](#6-skills技能扩展)
7. [AGENTS.md 自定义指令](#7-agentsmd-自定义指令)
8. [配置文件详解](#8-配置文件详解)
9. [非交互模式（CI/CD）](#9-非交互模式cicd)
10. [安全与沙箱](#10-安全与沙箱)
11. [使用场景参考](#11-使用场景参考)

---

## 1. Codex 是什么

Codex 是 OpenAI 推出的 **AI 编程 Agent**，可以理解代码、写代码、修 Bug、做代码审查，并通过自然语言指令完成完整的开发任务。

**核心能力：**

| 能力 | 说明 |
|------|------|
| 生成代码 | 描述需求，Codex 直接生成符合项目结构的代码 |
| 读懂代码 | 解释复杂逻辑，帮助理解陌生代码库 |
| 代码审查 | 识别 Bug、逻辑错误、边界情况 |
| 排查问题 | 追踪报错，给出精准修复建议 |
| 处理重复工作 | 重构、写测试、迁移代码 |

**适用计划：** ChatGPT Plus、Pro、Business、Edu、Enterprise

---

## 2. 快速上手

### 最快路径（5 分钟跑起来）

**推荐：先用 Desktop App**

1. 下载安装（macOS / Windows）
2. 用 ChatGPT 账号或 OpenAI API Key 登录
3. 选择一个本地项目文件夹
4. 直接发消息描述你的任务

> **新手建议：** 每次任务前后用 Git 打一个 checkpoint，方便随时撤销 Codex 的修改。

---

### 四种入口对比

| 方式 | 适合场景 | 平台 |
|------|----------|------|
| Desktop App | 日常开发，多项目切换 | macOS、Windows |
| IDE 扩展 | 不想离开编辑器 | VS Code、JetBrains |
| CLI | 深度用户，脚本自动化 | macOS、Windows、Linux |
| Cloud 云端 | 不占本地资源，连 GitHub | 浏览器 |

---

## 3. 四种使用方式

### 3.1 Desktop App（桌面应用）

**下载：** macOS（Apple Silicon / Intel）、Windows，Linux 即将支持

**核心功能：**

- 多项目线程并行，快速切换
- Git Worktree 集成，并行管理多分支改动
- 内置终端 + 文件对比工具
- 可直接 inspect diff、stage files、commit、push
- Computer Use：让 Codex 操控 macOS 应用
- 内置浏览器：处理页面渲染和浏览器流程
- 图片生成与编辑
- 定时任务和 thread 唤醒
- Plugin 系统（MCP Server）

**上手步骤：**
1. 下载安装 → 登录 → 选项目文件夹 → 发消息

---

### 3.2 IDE 扩展

**支持的编辑器：**

- VS Code（以及 Cursor、Windsurf）
- JetBrains 全家桶（Rider、IntelliJ、PyCharm、WebStorm）
- 平台：macOS、Windows、Linux

**安装：** VS Code Marketplace 搜索 `Codex` 安装，登录后侧边栏出现 Codex 面板

**功能亮点：**

- 利用当前打开的文件和选中内容作为上下文
- 三种 Approval 模式：Chat / Agent / Full-access Agent
- 推理力度可调：低 / 中 / 高
- 云端委托：复杂任务丢给云端跑，不占本地资源
- Slash 命令快速调整设置

---

### 3.3 CLI（命令行）

**安装：**

```bash
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"

# 或用 npm
npm install -g @openai/codex

# 或用 Homebrew
brew install openai-codex
```

**启动：**

```bash
codex                          # 进入交互式 TUI
codex "帮我解释一下这段代码"      # 带任务启动
codex --model gpt-5.5 "..."    # 指定模型
```

---

### 3.4 Cloud 云端版

**入口：** `chatgpt.com/codex`

**特点：**
- 连接 GitHub 仓库，任务在云端沙箱环境里运行
- 可监控进度，自动创建 Pull Request
- 不占用本地机器资源

---

## 4. CLI 核心功能详解

### 4.1 交互式 TUI

启动后进入全屏终端界面，支持：

- 实时发送提示，查看执行计划
- 逐步审批或拒绝每个操作步骤
- 常用快捷命令：
  - `/clear` — 清空对话
  - `/copy` — 复制最近回复
  - `/exit` — 退出

**键盘技巧：**

| 快捷键 | 功能 |
|--------|------|
| `@` + 文件名 | 模糊搜索引用文件 |
| `Ctrl+R` | 搜索历史提示词 |
| `Esc` | 编辑上一条消息 |
| `Tab`（任务运行时） | 排队下一条指令 |
| `!` + 命令 | 直接注入 shell 命令 |

---

### 4.2 会话管理

```bash
codex resume --last             # 恢复最近的会话
codex resume --all              # 显示所有历史会话（不过滤目录）
codex resume <session-id>       # 恢复指定会话
```

---

### 4.3 模型切换

```bash
codex --model gpt-5.5 "..."    # 启动时指定
```

或在会话中用 `/model` 命令随时切换。

**推荐模型：** `gpt-5.5`（复杂编程任务首选）

---

### 4.4 图片输入

```bash
codex -i screenshot.png "帮我实现这个 UI"
codex --image diagram.jpeg "根据这个架构图写代码"
```

支持 PNG、JPEG 等常见格式。

---

### 4.5 图片生成

在提示词中用自然语言描述，或使用 `$imagegen` 触发，底层使用 `gpt-image-2`。

---

### 4.6 代码审查

```bash
codex /review                   # 审查未提交的改动
codex /review --base main       # 对比 main 分支审查
```

不会修改工作区，只输出分析报告。

---

### 4.7 Web 搜索

默认使用**缓存索引**（降低 prompt injection 风险），需要实时结果时：

```bash
codex --search "..."            # 启用实时搜索
```

配置文件中可永久关闭：`web_search = "disabled"`

---

### 4.8 Approval 模式

| 模式 | 说明 |
|------|------|
| `untrusted`（默认） | 每步操作都需要审批 |
| `on-request` | 只在需要时询问 |
| `never` | 全自动，不询问（谨慎使用） |

---

### 4.9 远程 TUI 连接

在一台机器上运行 App Server，通过 WebSocket 在另一台机器操控：

- 支持 Capability Token 和 Signed Bearer Token 认证
- 适合远程开发、服务器场景

---

## 5. Slash 命令速查

在 CLI 交互模式中输入 `/` 触发命令菜单。

### 模型 & 性能

| 命令 | 说明 |
|------|------|
| `/model` | 切换 AI 模型 |
| `/fast` | 切换快速模式 |
| `/personality` | 调整回复风格 |

### 审批 & 安全

| 命令 | 说明 |
|------|------|
| `/permissions` | 调整操作审批规则 |
| `/approve` | 重试刚被拒绝的操作 |

### 会话管理

| 命令 | 说明 |
|------|------|
| `/clear` | 清空当前对话 |
| `/new` | 开始新会话 |
| `/fork` | 分支出并行会话 |
| `/resume` | 恢复历史会话 |
| `/compact` | 压缩长对话，节省上下文 |

### 信息 & 审查

| 命令 | 说明 |
|------|------|
| `/status` | 查看当前配置信息 |
| `/diff` | 查看 Git 改动 |
| `/review` | 请求代码分析 |
| `/goal` | 设置持续性任务目标 |

### 界面定制

| 命令 | 说明 |
|------|------|
| `/keymap` | 自定义键位绑定 |
| `/theme` | 切换配色主题 |
| `/statusline` | 调整状态栏 |
| `/title` | 设置会话标题 |

### 工具集成

| 命令 | 说明 |
|------|------|
| `/mcp` | 查看可用 MCP 工具 |
| `/plugins` | 管理插件 |
| `/apps` | 发现可用应用 |
| `/memories` | 管理记忆功能 |

> **注意：** 破坏性操作（如 `/clear`、`/exit`）需要二次确认；任务运行中可按 `Tab` 排队命令。

---

## 6. Skills（技能扩展）

Skills 是可复用的任务模块，将指令、资源、脚本打包在一起，扩展 Codex 的能力。

### 激活方式

- **显式调用：** `/skills` 命令或 `$技能名` 提及
- **隐式触发：** 任务描述匹配 Skill 描述时自动激活

### 目录结构

```
my-skill/
├── SKILL.md          # 必须，包含 name 和 description 元数据
├── scripts/          # 可选，可执行脚本
├── references/       # 可选，参考文档
├── assets/           # 模板和资源文件
└── agents/
    └── openai.yaml   # 可选，UI 元数据和依赖声明
```

### SKILL.md 示例

```markdown
---
name: code-reviewer
description: 对 PR 进行安全和性能审查
---

审查以下代码，重点关注：
1. SQL 注入风险
2. 性能 N+1 问题
3. 未处理的异常
```

### Skill 存放位置

| 作用域 | 路径 | 适用场景 |
|--------|------|----------|
| 项目本地 | `.agents/skills`（当前目录） | 项目专属工作流 |
| 项目共享 | 父目录 `.agents/skills` | 团队共享 |
| 仓库全局 | 仓库根目录 `.agents/skills` | 组织级共享 |
| 个人 | `~/.agents/skills` | 个人常用 |
| 系统级 | `/etc/codex/skills` | 系统管理员部署 |
| 内置 | Codex 自带 | 开箱即用 |

### 最佳实践

- 一个 Skill 只做一件事
- 优先用指令而非脚本（除非需要确定性行为）
- 用清晰的祈使句描述步骤，明确输入输出
- 广泛分发时打包成 Plugin

---

## 7. AGENTS.md 自定义指令

`AGENTS.md` 是给 Codex 的"工作约定文档"，在执行任务前自动读取，让 Codex 按照你的团队规范工作。

### 优先级（从高到低）

1. `~/.codex/AGENTS.override.md` — 全局临时覆盖
2. `~/.codex/AGENTS.md` — 全局默认
3. 从 Git 根目录到当前目录，逐层查找 `AGENTS.override.md`
4. 从 Git 根目录到当前目录，逐层查找 `AGENTS.md`

**合并规则：** 从根目录往下拼接，越靠近当前目录优先级越高。

---

### 创建全局指令

```bash
cat > ~/.codex/AGENTS.md << 'EOF'
# 工作约定

## 代码风格
- 使用 TypeScript，禁止 any
- 函数必须有明确的返回类型
- 提交信息用英文

## 测试要求
- 修 Bug 先写复现测试
- 覆盖率目标 80%

## 安全规则
- 密钥只用环境变量，不硬编码
EOF
```

---

### 项目级指令

在项目根目录创建 `.codex/AGENTS.md`（或直接 `AGENTS.md`）：

```markdown
# 项目约定

## 架构规则
- 数据库操作只在 repository 层
- API 响应统一用 ApiResponse<T> 包装

## 禁止操作
- 不要修改 migrations/ 目录
- 不要删除 legacy/ 下的文件
```

---

### 验证配置是否生效

```bash
codex --ask-for-approval never "总结一下当前的指令配置"
```

---

### 配置自定义文件名

在 `~/.codex/config.toml` 中：

```toml
project_doc_fallback_filenames = ["CLAUDE.md", "CURSOR_RULES.md"]
project_doc_max_bytes = 65536   # 默认 32KB，可调大
```

---

## 8. 配置文件详解

### 配置层级（优先级从高到低）

1. CLI flags（`--model`、`-c key=value`）
2. 项目配置（最近目录的 `.codex/config.toml`）
3. 命名 Profile 文件
4. 用户配置（`~/.codex/config.toml`）
5. 系统配置（`/etc/codex/config.toml`）
6. 内置默认值

---

### 基础配置（`~/.codex/config.toml`）

```toml
# 默认模型
model = "gpt-5.5"

# 审批策略：untrusted（每步审批）| on-request（按需）| never（全自动）
approval_policy = "on-request"

# 沙箱模式：read-only | workspace-write | danger-full-access
sandbox_mode = "workspace-write"

# Web 搜索：cached | live | disabled
web_search = "cached"

# 推理力度：low | medium | high
model_reasoning_effort = "high"

# Windows 用户专属
[windows]
sandbox = "elevated"
```

---

### 功能开关（Feature Flags）

```toml
[features]
memories = true           # 启用记忆功能
shell_tools = true        # 启用 shell 工具
personality = true        # 启用个性化风格
child_agents_md = true    # 启用层级 AGENTS.md
```

---

### 自定义模型 Provider

```toml
[[providers]]
id = "my-provider"
base_url = "https://api.example.com/v1"
auth_method = "env_var"
auth_env_var = "MY_API_KEY"
```

---

### 命名 Profile（多场景切换）

```bash
# 创建 deep-review profile
cat > ~/.codex/deep-review.config.toml << 'EOF'
model = "gpt-5.5"
model_reasoning_effort = "high"
approval_policy = "untrusted"
EOF

# 使用指定 profile
codex --profile deep-review "审查这个 PR"
```

---

### 临时覆盖配置

```bash
codex -c model=gpt-5.5 -c approval_policy=never "..."
```

---

### MCP Server 配置

```toml
[[mcp_servers]]
id = "my-server"
type = "stdio"
command = ["npx", "my-mcp-server"]
env = { MY_TOKEN = "$MY_TOKEN" }
startup_timeout_ms = 10000
```

---

### Hooks（生命周期钩子）

```toml
[hooks]
pre_tool_call = ["sh", "-c", "echo 'about to run a tool'"]
```

---

### 可观测性（OpenTelemetry）

```toml
[otel]
endpoint = "http://localhost:4318"
enabled = true

[analytics]
enabled = false   # 关闭匿名分析数据
```

---

## 9. 非交互模式（CI/CD）

`codex exec` 让 Codex 在脚本和 CI 流水线中运行，不打开交互界面。

### 基础用法

```bash
# 基本任务
codex exec "总结仓库结构并列出最高风险的 5 个模块"

# 结果重定向
codex exec "列出所有 TODO 注释" > todos.txt

# 带写入权限
codex exec --sandbox workspace-write "修复所有 ESLint 错误"
```

---

### 实用场景

```bash
# CI 自动修复 lint 错误
codex exec --sandbox workspace-write "修复所有 TypeScript 类型错误"

# 分析构建日志
cat build.log | codex exec "分析这份构建日志，找出根本原因"

# 自动生成 PR 描述
git diff main | codex exec "根据这个 diff 起草 PR 描述"

# 定时代码审查
codex exec "审查过去 24 小时的提交，找出潜在风险"
```

---

### JSON 结构化输出

```bash
# 输出 JSON Lines 格式（每个事件一行 JSON）
codex exec --json "分析代码质量"

# 指定输出 Schema
codex exec --output-schema schema.json "提取所有 API 接口"
```

---

### 安全权限

| 权限级别 | 说明 |
|----------|------|
| 默认（read-only） | 只读，不能修改文件 |
| `--sandbox workspace-write` | 可以编辑工作区文件 |
| `--sandbox danger-full-access` | 完全访问（仅限受控环境） |

---

### 认证方式

```bash
# 临时使用指定 API Key
CODEX_API_KEY=sk-xxx codex exec "..."

# 恢复上次会话继续任务
codex exec resume --last "继续刚才的重构任务"
```

---

### 忽略本地配置（标准化 CI 环境）

```bash
codex exec --ignore-user-config --ignore-rules "..."
```

---

## 10. 安全与沙箱

### 沙箱模式

Codex 默认在沙箱中运行，限制文件系统和网络访问，防止意外操作。

| 沙箱模式 | 说明 |
|----------|------|
| `read-only` | 只能读文件，不能修改 |
| `workspace-write` | 可修改工作区文件（推荐日常使用） |
| `danger-full-access` | 完全访问，慎用 |

---

### Codex Security（安全扫描）

Codex Security 是面向工程和安全团队的漏洞扫描功能：

- 逐 commit 扫描 GitHub 仓库，建立代码上下文
- 在隔离环境中验证漏洞（减少误报）
- 输出优先级排序的漏洞列表，附修复建议

**要求：** 通过 Codex Web 连接 GitHub 仓库，需 OpenAI 托管访问权限。

---

### Approval 工作流

任何"有风险"的操作（写文件、运行命令、网络请求）都会在执行前等待你的确认：

- `untrusted`：每步都问（最安全）
- `on-request`：Codex 认为需要时问
- `never`：全自动执行（适合 CI 场景）

---

### 企业级管理（`requirements.toml`）

管理员可通过 `/etc/codex/requirements.toml` 强制统一配置：

```toml
# 禁止用户覆盖 hooks 配置
allow_managed_hooks_only = true

# 强制指定模型
model = "gpt-5.5"
```

---

## 11. 使用场景参考

### 工程类

| 场景 | 示例提示词 |
|------|-----------|
| 功能开发 | "在用户模块里加一个邮箱验证功能，参考现有的手机验证逻辑" |
| Bug 修复 | "登录接口偶发 500，帮我排查 auth/login.ts" |
| 代码重构 | "把 UserService 里的数据库查询提取到 Repository 层" |
| 写测试 | "给 PaymentService 的 processRefund 方法补全单测" |
| 代码迁移 | "把所有 callback 风格的代码改成 async/await" |

### 代码审查类

| 场景 | 示例提示词 |
|------|-----------|
| PR 审查 | `/review --base main` |
| 安全扫描 | "检查这个文件有没有 SQL 注入风险" |
| 性能分析 | "找出这段代码里的 N+1 查询" |

### 自动化类

| 场景 | 命令 |
|------|------|
| CI 自动修 lint | `codex exec --sandbox workspace-write "fix all ESLint errors"` |
| 每日代码健康报告 | `codex exec "生成今日代码质量报告" > report.md` |
| 自动生成 PR 描述 | `git diff main \| codex exec "写 PR 描述"` |

---

## 附录：常用命令速查

```bash
# 安装
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# 启动交互式 TUI
codex

# 带任务启动
codex "帮我重构 auth 模块"

# 指定模型
codex --model gpt-5.5 "..."

# 恢复上次会话
codex resume --last

# 非交互执行
codex exec "总结仓库结构"

# 代码审查
codex exec "审查最近的改动" 

# 查看所有配置项
codex config --help
```

---

## 相关资源

- [官方文档](https://developers.openai.com/codex)
- [GitHub 官方 Repo](https://github.com/openai/codex)
- [Awesome Codex CLI](https://github.com/RoggeOhta/awesome-codex-cli) — 150+ 工具、Skills、插件合集
- [Codex Cheat Sheet](https://github.com/BA-CalderonMorales/codex-cheat-sheet) — 英文快速参考
- [OpenAI Changelog](https://developers.openai.com/codex/changelog) — 最新更新记录
