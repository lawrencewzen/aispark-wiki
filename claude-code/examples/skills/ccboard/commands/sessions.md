> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: sessions
description: 浏览 Claude Code 会话历史
category: exploration
---

# 会话浏览命令

启动 ccboard 并直接跳转到会话探索标签页。

## 功能

- **项目树**：浏览 33+ 个项目的嵌套结构
- **会话列表**：1200+ 条会话及其元数据
- **搜索**：按项目、消息或模型过滤会话（按 `/` 激活）
- **会话详情**：
  - 时间戳（开始、结束、时长）
  - Token 用量分项
  - 使用的模型
  - 首条消息预览
- **文件操作**：
  - `e` ：在编辑器中打开会话 JSONL 文件
  - `o` ：在访达中显示会话文件

## 用法

```bash
# 直接打开会话标签页
/sessions

# 备选：带标签页参数运行
ccboard --tab sessions
```

## 会话标签页导航

- `←/→` ：在项目树和会话列表之间切换
- `↑/↓` ：导航列表项
- `Enter` ：查看会话详情
- `/` ：打开搜索输入框
- `e` ：编辑选中的会话 JSONL 文件
- `o` ：显示会话文件

## 会话元数据

每条会话显示：
- **ID**：唯一会话标识符
- **Started**：首条消息的时间戳
- **Duration**：对话总时长
- **Messages**：消息条数
- **Tokens**：消耗的总 token 数
- **Models**：使用的 AI 模型（如 opus-4.5、sonnet-4.5）
- **Preview**：首条用户消息（最多 200 个字符）

## 搜索示例

```
# 按项目名称搜索
/my-project

# 按模型搜索
/opus

# 按消息内容搜索
/implement feature
```

## 前提条件

需要已安装 ccboard。如未安装，请运行 `/ccboard-install`。

## 实现

```bash
#!/bin/bash

# Check if ccboard is installed
if ! command -v ccboard &> /dev/null; then
    echo "❌ ccboard is not installed"
    echo "Run: /ccboard-install"
    exit 1
fi

# Launch ccboard with Sessions tab (tab index 1, accessible with '2' key)
# For now, launch and user presses '2'
exec ccboard
```

**注意**：当前以仪表盘视图启动 ccboard，按 `2` 键进入会话标签页。
未来版本将支持 `ccboard --tab sessions` 直接跳转。
