> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ccboard-web
description: 启动 ccboard Web 界面
category: monitoring
---

# Web 界面命令

启动 ccboard Web UI，通过浏览器进行监控和可视化。

## 功能特性

- **Web 仪表板**：在任意浏览器中访问 ccboard
- **实时更新**：通过 Server-Sent Events（SSE）推送实时数据
- **响应式设计**：支持桌面、平板、移动端
- **数据共享**：与 TUI 共用同一数据层（单一二进制）
- **并发访问**：多用户可同时查看

## 使用方式

```bash
# 在默认端口 3333 启动 Web UI
/ccboard-web

# 或指定自定义端口
ccboard web --port 8080
```

## 访问地址

启动后，在浏览器中打开：
```
http://localhost:3333
```

## 运行模式

ccboard 支持 3 种运行模式：

1. **仅 TUI**（默认）：
   ```bash
   ccboard
   ```

2. **仅 Web**：
   ```bash
   ccboard web --port 3333
   ```

3. **同时运行**：
   ```bash
   ccboard both --port 3333
   ```
   在终端运行 TUI，同时在端口 3333 启动 Web 服务器

## Web UI 功能

- 带实时统计的仪表板
- 带分页的会话浏览器
- 配置查看器（只读）
- 钩子、智能体、费用可视化
- MCP 服务器状态
- 历史记录与搜索

## 前置条件

必须已安装 ccboard。如未安装，请运行 `/ccboard-install`。

## 实现脚本

```bash
#!/bin/bash

# 检查 ccboard 是否已安装
if ! command -v ccboard &> /dev/null; then
    echo "❌ ccboard is not installed"
    echo "Run: /ccboard-install"
    exit 1
fi

# 默认端口
PORT="${1:-3333}"

echo "🌐 Launching ccboard web interface..."
echo "Access at: http://localhost:$PORT"
echo ""
echo "Press Ctrl+C to stop"
echo ""

# 启动 Web UI
exec ccboard web --port "$PORT"
```

**注意**：Web UI 目前仍在开发中，TUI 是功能完整的主要界面。
