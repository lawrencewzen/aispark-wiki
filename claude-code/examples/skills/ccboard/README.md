> 📚 **AI Spark Wiki** · Claude Code 知识库

# ccboard - Claude Code 控制台插件

> 用于监控和管理 Claude Code 的综合 TUI/Web 控制台

[![License](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](../../../LICENSE)
[![Rust](https://img.shields.io/badge/rust-1.70%2B-orange.svg)](https://www.rust-lang.org)

## 快速开始

### 安装

```bash
# 使用 Claude Code 命令
/ccboard-install

# 或通过 cargo 手动安装
cargo install ccboard
```

### 启动控制台

```bash
# 启动 TUI
/dashboard

# 或直接运行
ccboard
```

## 命令

| 命令 | 说明 |
|---------|-------------|
| `/dashboard` | 启动交互式 TUI 控制台 |
| `/mcp-status` | 监控 MCP 服务器（按 `8`） |
| `/costs` | 查看费用分析（按 `6`） |
| `/sessions` | 浏览对话历史（按 `2`） |
| `/ccboard-web` | 启动 Web 界面 |
| `/ccboard-install` | 安装或更新 ccboard |

## 功能

- **8 个交互标签页**：控制台、会话、配置、钩子、智能体、费用、历史、MCP
- **实时监控**：文件监听器提供即时更新
- **MCP 管理**：服务器状态与配置
- **费用追踪**：Token 用量与定价分析（如总计 $9,145）
- **会话浏览器**：跨 33+ 个项目浏览 1,200+ 次对话
- **文件编辑**：按 `e` 在 $EDITOR 中编辑文件
- **双界面**：单一二进制文件同时提供终端（TUI）和 Web UI

## 导航

**跳转到标签页**：
- `1` 控制台
- `2` 会话
- `3` 配置
- `4` 钩子
- `5` 智能体
- `6` 费用
- `7` 历史
- `8` MCP

**常用快捷键**：
- `Tab` / `Shift+Tab`：切换标签页
- `e`：在编辑器中打开文件
- `o`：在访达中显示文件
- `q`：退出
- `F5`：刷新

## MCP 服务器监控

MCP 标签页（按 `8`）提供：

- **实时状态**：● 运行中、○ 已停止、? 未知
- **服务器详情**：完整命令、参数、环境变量
- **快捷操作**：
  - `e`：编辑 `claude_desktop_config.json`
  - `o`：在访达中显示配置
  - `r`：刷新服务器状态

## 费用分析

追踪您的 Claude Code 支出：

- 总 Token：17.32M
- 总费用：$9,145.20
- 按模型细分：Opus 4.5（76%）、Sonnet 4.5（14%）
- 缓存命中率：99.9%

## 会话浏览器

浏览和搜索对话：

- 跨 33+ 个项目的 1,200+ 次会话
- 全文搜索（按 `/`）
- 元数据：时间戳、Token 数、模型信息
- 直接编辑 JSONL 文件

## Web 界面

```bash
# 启动 Web UI
/ccboard-web

# 或使用自定义端口
ccboard web --port 8080

# 同时运行 TUI 和 Web
ccboard both --port 3333
```

## 系统要求

- **Rust 1.70+** 及 Cargo
- 已安装 **Claude Code**（从 `~/.claude/` 读取数据）

## 架构

单一 Rust 二进制文件（2.4MB），包含：
- **TUI**：基于 Ratatui 的终端界面
- **Web**：Axum + Leptos Web 界面
- **核心**：带文件监听器的共享数据层

## 数据来源

ccboard 从以下路径读取数据：
- `~/.claude/stats-cache.json` - 统计信息
- `~/.claude/claude_desktop_config.json` - MCP 配置
- `~/.claude/projects/*/` - 会话 JSONL 文件
- `.claude/settings.json` - 配置文件

**只读模式**：非侵入式监控，与 Claude Code 同时运行安全无虞。

## 性能

- 初始加载：1,000+ 会话不超过 2 秒
- 内存：典型使用约 50MB
- 懒加载：会话内容按需加载

## 局限性

当前版本（0.1.0）：

- **只读模式**：不支持写操作
- **MCP 状态**：仅支持 Unix（macOS/Linux）
- **Web UI**：开发中

## 故障排除

### 找不到 ccboard
```bash
which ccboard          # 检查是否已安装
/ccboard-install       # 如未安装则执行安装
```

### 无数据显示
```bash
ls ~/.claude/          # 验证 Claude Code 目录
cat ~/.claude/stats-cache.json  # 检查统计文件
```

### MCP 状态显示"未知"
- 需要 Unix（macOS/Linux）
- Windows 默认显示"Unknown"
- 验证服务器是否运行：`ps aux | grep <server-name>`

## 文档

- **完整指南**：参见 [SKILL.md](SKILL.md) 获取完整文档
- **命令**：参见 [commands/](commands/) 目录
- **脚本**：参见 [scripts/](scripts/) 目录

## 链接

- **代码库**：https://github.com/{OWNER}/ccboard
- **问题反馈**：https://github.com/{OWNER}/ccboard/issues
- **Claude Code**：https://claude.ai/code

## 许可证

MIT OR Apache-2.0

---

**为 Claude Code 社区用心打造**
