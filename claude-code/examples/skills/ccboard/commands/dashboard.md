> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: dashboard
description: 启动 ccboard TUI 看板
category: monitoring
---

# Dashboard 命令

启动交互式 ccboard TUI，可视化监控你的 Claude Code 使用情况。

## 功能特性

- **8 个交互标签页**：Dashboard、Sessions、Config、Hooks、Agents、Costs、History、MCP
- **实时监控**：文件监视器，支持实时更新
- **MCP 管理**：服务器状态与配置
- **费用追踪**：Token 用量与定价分析
- **会话浏览器**：浏览和搜索历史对话
- **文件编辑**：按 `e` 键在 $EDITOR 中编辑文件

## 使用方式

```bash
# 启动 TUI 看板
/dashboard

# 或直接运行
ccboard
```

## 导航操作

- `1-8` ：跳转到指定标签页
- `Tab` / `Shift+Tab` ：切换标签页
- `q` ：退出
- `F5` ：刷新数据
- `e` ：编辑选中文件
- `o` ：在访达中显示文件

## 前置要求

需要先安装 ccboard。如未安装，请运行：
```bash
/ccboard-install
```

## 实现代码

```bash
#!/bin/bash

# 检查 ccboard 是否已安装
if ! command -v ccboard &> /dev/null; then
    echo "❌ ccboard is not installed"
    echo ""
    echo "Install with:"
    echo "  /ccboard-install"
    echo ""
    echo "Or manually:"
    echo "  cargo install ccboard"
    exit 1
fi

# 启动 ccboard TUI
exec ccboard
```
