> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: mcp-status
description: 打开 ccboard MCP 服务器标签页
category: monitoring
---

# MCP Status 命令

启动 ccboard 并直接跳转到 MCP 服务器管理标签页。

## 功能特性

- **服务器列表**：显示 `claude_desktop_config.json` 中配置的所有 MCP 服务器
- **状态检测**：实时状态（● 运行中、○ 已停止、? 未知）
- **服务器详情**：完整命令、参数、环境变量
- **快捷操作**：
  - `e` ：编辑 MCP 配置
  - `o` ：在访达中显示配置文件
  - `r` ：刷新服务器状态

## 使用方式

```bash
# 直接打开 MCP 标签页
/mcp-status

# 或带参数运行
ccboard --tab mcp
```

## MCP 标签页导航

- `h/j/k/l` 或 `←/→/↑/↓` ：导航
- `Enter` ：聚焦详情面板
- `e` ：编辑 `~/.claude/claude_desktop_config.json`
- `o` ：显示配置文件位置
- `r` ：刷新服务器状态

## 服务器状态

- **● 绿色** ：服务器进程运行中
- **○ 红色** ：服务器进程已停止
- **? 灰色** ：状态未知（Windows 或检测失败）

## 前置要求

需要先安装 ccboard。如未安装，请运行 `/ccboard-install`。

## 实现代码

```bash
#!/bin/bash

# 检查 ccboard 是否已安装
if ! command -v ccboard &> /dev/null; then
    echo "❌ ccboard is not installed"
    echo "Run: /ccboard-install"
    exit 1
fi

# 启动 ccboard 并进入 MCP 标签页（标签页索引 7，按 '8' 键访问）
# 目前先启动，用户按 '8' 进入 MCP 标签页
# TODO: 未来版本将在 ccboard CLI 中添加 --tab 参数
exec ccboard
```

**注意**：当前以看板视图启动 ccboard。按 `8` 键进入 MCP 标签页。
未来版本将支持 `ccboard --tab mcp` 直接访问。
