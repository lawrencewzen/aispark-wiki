> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: costs
description: 打开 ccboard 费用分析标签页
category: analytics
---

# 费用分析命令

启动 ccboard 并直接跳转到费用追踪与分析标签页。

## 功能

- **3 个视图**：
  - 概览：总费用及分项明细
  - 按模型：各 AI 模型费用（Opus、Sonnet、Haiku）
  - 每日趋势：费用随时间的变化曲线

- **Token 分项明细**：
  - 输入 token（提示词）
  - 输出 token（生成内容）
  - 缓存读取 token（复用内容）
  - 缓存写入 token（存储内容）

- **定价**：基于 Anthropic 2024 年费率自动计算

## 用法

```bash
# 直接打开费用标签页
/costs

# 备选：带标签页参数运行
ccboard --tab costs
```

## 费用标签页导航

- `1` ：概览视图
- `2` ：按模型视图
- `3` ：每日趋势视图
- `Tab` ：切换视图
- `↑/↓` ：滚动数据

## 示例输出

```
Total Tokens: 17.32M
Total Cost: $9,145.20

Breakdown by Model:
- Opus 4.5:   76% ($7,828)  ████████████████
- Sonnet 4.5: 14% ($1,314)  ███
- Haiku 3.5:  10% ($3)      █

Token Distribution:
- Input:       10.70M (65%)
- Output:      4.58M  (28%)
- Cache Read:  1.01M  (6%)
- Cache Write: 1.04B  (1%)
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

# Launch ccboard with Costs tab (tab index 5, accessible with '6' key)
# For now, launch and user presses '6'
exec ccboard
```

**注意**：当前以仪表盘视图启动 ccboard，按 `6` 键进入费用标签页。
未来版本将支持 `ccboard --tab costs` 直接跳转。
