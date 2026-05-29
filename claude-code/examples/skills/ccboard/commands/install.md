> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ccboard-install
description: 安装或更新 ccboard
category: setup
---

# 安装 ccboard 命令

通过 cargo 安装或更新 ccboard 二进制文件。

## ccboard 是什么？

ccboard 是一个功能完整的 TUI/Web 仪表板，用于监控和管理 Claude Code：

- **8 个交互式标签页**：Dashboard、Sessions、Config、Hooks、Agents、Costs、History、MCP
- **实时监控**：文件监听器实时推送更新
- **MCP 管理**：服务器状态与配置管理
- **费用分析**：Token 用量与费用追踪
- **会话浏览器**：浏览和搜索对话历史
- **双界面**：终端（TUI）和 Web UI

## 环境要求

- **Rust**：1.70 或更高版本
- **Cargo**：Rust 包管理器（随 Rust 一起安装）

如果尚未安装 Rust：
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## 用法

```bash
# 安装 ccboard
/ccboard-install

# 或手动安装
cargo install ccboard
```

## 安装流程

1. 检查 cargo 是否已安装
2. 检测是否已有 ccboard 安装
3. 若已安装，提示是否确认更新
4. 通过 `cargo install ccboard --force` 执行安装
5. 验证安装并显示版本号

## 安装后使用

安装完成后，可使用以下命令：

- `/dashboard` - 启动 TUI 仪表板
- `/mcp-status` - 打开 MCP 服务器标签页
- `/costs` - 打开费用分析
- `/sessions` - 浏览会话历史
- `/ccboard-web` - 启动 Web 界面

也可以直接运行：
```bash
ccboard              # 启动 TUI
ccboard web          # 启动 Web UI
ccboard --help       # 显示所有选项
```

## 故障排查

### 找不到 cargo
```bash
# 安装 Rust 和 cargo
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 重新加载 shell 环境
source $HOME/.cargo/env
```

### 安装失败
```bash
# 更新 Rust 工具链
rustup update

# 从源码手动安装
git clone https://github.com/{OWNER}/ccboard
cd ccboard
cargo install --path crates/ccboard
```

### 权限被拒绝
```bash
# 确保 ~/.cargo/bin 已加入 PATH
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 实现

```bash
#!/bin/bash

# 执行安装脚本
exec "$(dirname "$0")/../scripts/install-ccboard.sh"
```

## 卸载

要移除 ccboard：
```bash
cargo uninstall ccboard
```
