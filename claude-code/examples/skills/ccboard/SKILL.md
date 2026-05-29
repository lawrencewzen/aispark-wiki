> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ccboard
description: "启动并导航 ccboard TUI/Web 仪表板，用于 Claude Code 监控。适用于查看 Token 用量、追踪成本、浏览会话，以及检查各项目的 MCP 服务器状态。"
allowed-tools: Bash
effort: low
metadata:
  version: 0.1.0
---

# ccboard - Claude Code 仪表板

用于监控和管理 Claude Code 使用情况的综合 TUI/Web 仪表板。

## 概述

ccboard 提供统一界面，可视化并探索所有 Claude Code 数据：

- **会话（Sessions）**：浏览跨项目的所有对话
- **统计（Statistics）**：实时 Token 用量、缓存命中率、活跃度趋势
- **MCP 服务器**：监控和管理 Model Context Protocol 服务器
- **成本（Costs）**：通过详细 Token 明细和定价追踪支出
- **配置（Configuration）**：查看级联设置（全局 > 项目 > 本地）
- **钩子（Hooks）**：探索执行前/后钩子及自动化规则
- **智能体（Agents）**：管理自定义智能体、命令和技能
- **历史（History）**：跨所有消息的全文搜索

## 安装

### 通过 Cargo 安装（推荐）

```bash
# 使用 Claude Code 命令
/ccboard-install

# 或手动安装
cargo install ccboard
```

### 环境要求

- Rust 1.70+ 及 Cargo
- 已安装 Claude Code（从 `~/.claude/` 读取数据）

## 命令

| 命令 | 说明 | 快捷键 |
|---------|-------------|----------|
| `/dashboard` | 启动 TUI 仪表板 | `ccboard` |
| `/mcp-status` | 打开 MCP 服务器选项卡 | 按 `8` |
| `/costs` | 打开成本分析 | 按 `6` |
| `/sessions` | 浏览会话 | 按 `2` |
| `/ccboard-web` | 启动 Web 界面 | `ccboard web` |
| `/ccboard-install` | 安装/更新 ccboard | - |

## 功能

### 8 个交互式选项卡

#### 1. 仪表板（按 `1`）
- Token 用量统计
- 会话数量
- 已发送消息数
- 缓存命中率
- MCP 服务器数量
- 7 天活跃度迷你图
- 前 5 个模型的用量仪表盘

#### 2. 会话（按 `2`）
- 双窗格：项目树 + 会话列表
- 元数据：时间戳、持续时长、Token 数、模型
- 搜索：按项目、消息或模型过滤（按 `/`）
- 文件操作：`e` 编辑 JSONL，`o` 在 Finder 中显示

#### 3. 配置（按 `3`）
- 4 列级联视图：全局 | 项目 | 本地 | 合并结果
- 配置继承关系可视化
- MCP 服务器配置
- 规则（CLAUDE.md）预览
- 权限、钩子、环境变量
- 按 `e` 键编辑配置

#### 4. 钩子（按 `4`）
- 基于事件的钩子浏览（PreToolUse、UserPromptSubmit）
- 钩子 bash 脚本预览
- 匹配模式与条件
- 文件路径追踪，便于编辑

#### 5. 智能体（按 `5`）
- 3 个子选项卡：智能体（12）| / 命令（5）| ★ 技能（0）
- Frontmatter 元数据提取
- 文件预览与编辑
- 递归目录扫描

#### 6. 成本（按 `6`）
- 3 个视图：概览 | 按模型 | 日趋势
- Token 明细：输入、输出、缓存读/写
- 定价：总估算成本
- 模型分布明细

#### 7. 历史（按 `7`）
- 跨所有会话的全文搜索
- 按小时活跃度直方图（24 小时）
- 7 天迷你图
- 所有消息均可搜索

#### 8. MCP（按 `8`）**新增**
- 双窗格：服务器列表（35%）| 详情（65%）
- 实时状态检测：● 运行中、○ 已停止、? 未知
- 完整服务器详情：命令、参数、环境变量
- 快捷操作：`e` 编辑配置，`o` 显示文件，`r` 刷新状态

### 导航

**全局快捷键**：
- `1-8` ：跳转到对应选项卡
- `Tab` / `Shift+Tab` ：切换选项卡
- `q` ：退出
- `F5` ：刷新数据

**Vim 风格**：
- `h/j/k/l` ：导航（左/下/上/右）
- `←/→/↑/↓` ：方向键替代方案

**常用操作**：
- `Enter` ：查看详情 / 聚焦窗格
- `e` ：在 $EDITOR 中编辑文件
- `o` ：在 Finder 中显示文件
- `/` ：搜索（在会话/历史选项卡中）
- `Esc` ：关闭弹窗 / 取消

### 实时监控

ccboard 内置文件监听器，持续监控 `~/.claude/` 的变化：

- **统计更新**：`stats-cache.json` 变化时实时刷新
- **会话更新**：新会话自动出现
- **配置更新**：设置变更实时反映到界面
- **500ms 防抖**：防止过于频繁的更新

### 文件编辑

在任意条目上按 `e` 可在首选编辑器中打开：

- 优先级：`$VISUAL` > `$EDITOR` > 平台默认（nano/notepad）
- 支持：会话（JSONL）、配置（JSON）、钩子（Shell）、智能体（Markdown）
- 保留终端状态（备用屏幕模式）
- 跨平台支持（macOS、Linux、Windows）

### MCP 服务器管理

MCP 选项卡提供全面的服务器监控：

**状态检测**（Unix）：
- 通过 `ps aux` 检查运行中的进程
- 从命令中提取包名
- 运行时显示 PID
- Windows 显示"未知"状态

**服务器详情**：
- 完整命令及参数
- 环境变量及其值
- 配置文件路径（`~/.claude/claude_desktop_config.json`）
- 快速编辑/显示操作

**导航**：
- `h/l` 或 `←/→` ：在列表与详情之间切换
- `j/k` 或 `↑/↓` ：选择服务器
- `Enter` ：聚焦详情窗格
- `e` ：编辑 MCP 配置
- `o` ：在 Finder 中显示配置
- `r` ：刷新服务器状态

## 使用示例

### 日常监控

```bash
# 启动仪表板
/dashboard

# 查看活跃度和成本
# 按 '1' 查看概览
# 按 '6' 查看成本明细
# 按 '7' 查看最近历史
```

### MCP 故障排查

```bash
# 打开 MCP 选项卡
/mcp-status

# 或：运行 ccboard 后按 '8'

# 检查服务器状态（● 绿色 = 运行中）
# 如需要，按 'e' 编辑配置
# 修改后按 'r' 刷新状态
```

### 会话分析

```bash
# 浏览会话
/sessions

# 按 '/' 进行搜索
# 按项目过滤：/my-project
# 按模型过滤：/opus
# 在会话上按 'e' 查看完整 JSONL
```

### 成本追踪

```bash
# 查看成本
/costs

# 按 '1' 查看概览
# 按 '2' 查看按模型明细
# 按 '3' 查看日趋势

# 找出高消耗会话
# 追踪缓存效率（99.9% 命中率）
```

## Web 界面

启动基于浏览器的界面，用于远程监控：

```bash
# 启动 Web 界面
/ccboard-web

# 或指定自定义端口
ccboard web --port 8080

# 访问地址：http://localhost:3333
```

**功能**：
- 与 TUI 数据相同（共享后端）
- 通过 Server-Sent Events（SSE）实现实时更新
- 响应式设计（桌面/平板/移动端）
- 支持多用户并发访问

**同时运行两种界面**：
```bash
ccboard both --port 3333
```

## 架构

ccboard 是一个包含双前端的单一 Rust 二进制文件：

```
ccboard/
├── ccboard-core/      # 解析器、模型、数据存储、文件监听
├── ccboard-tui/       # Ratatui 前端（8 个选项卡）
└── ccboard-web/       # Axum + Leptos 前端
```

**数据来源**：
- `~/.claude/stats-cache.json` - 统计数据
- `~/.claude/claude_desktop_config.json` - MCP 配置
- `~/.claude/projects/*/` - 会话 JSONL 文件
- `~/.claude/settings.json` - 全局设置
- `.claude/settings.json` - 项目设置
- `.claude/settings.local.json` - 本地覆盖配置
- `.claude/CLAUDE.md` - 规则与行为定义

## 故障排查

### 找不到 ccboard

```bash
# 检查安装情况
which ccboard

# 如需安装
/ccboard-install
```

### 无数据显示

```bash
# 确认 Claude Code 已安装
ls ~/.claude/

# 检查统计文件是否存在
cat ~/.claude/stats-cache.json

# 指定项目运行
ccboard --project ~/path/to/project
```

### MCP 状态显示"未知"

- 状态检测需要 Unix 系统（macOS/Linux）
- Windows 默认显示"未知"
- 检查服务器进程是否确实在运行：`ps aux | grep <server-name>`

### 文件监听器不工作

- 确认 `notify` crate 支持当前平台
- 检查 `~/.claude/` 的文件权限
- 若文件系统事件丢失，重启 ccboard

## 高级用法

### 命令行选项

```bash
ccboard --help              # 显示所有选项
ccboard --claude-home PATH  # 自定义 Claude 目录
ccboard --project PATH      # 指定项目
ccboard stats               # 打印统计信息后退出
ccboard web --port 8080     # 在 8080 端口启动 Web 界面
ccboard both                # 同时运行 TUI + Web
```

### 环境变量

```bash
# 编辑器偏好
export EDITOR=vim
export VISUAL=code

# 自定义 Claude 主目录
export CLAUDE_HOME=~/custom/.claude
```

### 与 Claude Code 集成

ccboard 以**只读**方式从 Claude Code 目录读取数据：

- 非侵入式监控
- 不修改 Claude 数据
- 可与 Claude Code 安全并发运行
- 文件监听器实时检测变化

## 性能

- **二进制大小**：2.4MB（Release 构建）
- **初始加载**：1000+ 会话在 2 秒内完成
- **内存占用**：典型使用约 50MB
- **CPU 占用**：监控过程中低于 5%
- **懒加载**：会话内容按需加载

## 已知限制

当前版本（0.1.0）：

- **只读**：不支持对 Claude 数据的写操作
- **MCP 状态**：仅支持 Unix（Windows 显示"未知"）
- **Web 界面**：仍在开发中（TUI 为主要界面）
- **搜索**：基础子字符串匹配（暂不支持模糊搜索）

未来规划：

- 增强 MCP 服务器管理（启动/停止）
- MCP 协议健康检查
- 导出报告（PDF、JSON、CSV）
- 配置编辑（写入 settings.json）
- 会话恢复集成
- 增强搜索（支持模糊匹配）

## 参与贡献

ccboard 是开源项目（MIT OR Apache-2.0）。

仓库地址：https://github.com/{OWNER}/ccboard

欢迎以下形式的贡献：
- 提交 Bug 报告和功能请求
- 提交新功能 PR
- 改进文档
- 平台特定测试（Windows、Linux）

## 致谢

构建所用技术：
- [Ratatui](https://ratatui.rs/) - 终端 UI 框架
- [Axum](https://github.com/tokio-rs/axum) - Web 框架
- [Leptos](https://leptos.dev/) - 响应式前端
- [Notify](https://github.com/notify-rs/notify) - 文件监听器
- [Serde](https://serde.rs/) - 序列化

## 许可证

MIT OR Apache-2.0

---

**有疑问？**

- GitHub Issues：https://github.com/{OWNER}/ccboard/issues
- 文档：https://github.com/{OWNER}/ccboard
- Claude Code：https://claude.ai/code
