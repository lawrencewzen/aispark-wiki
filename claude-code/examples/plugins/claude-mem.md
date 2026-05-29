> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "claude-mem 插件模板"
description: "自动持久化记忆插件，跨会话捕获工具调用和决策记录"
tags: [plugin, memory, integration]
---

# claude-mem 插件模板

**用途**：在 Claude Code 会话之间自动持久化记忆
**仓库**：https://github.com/thedotmack/claude-mem
**类型**：官方插件（26.5k+ stars）
**版本**：v10.6.3
**许可证**：AGPL-3.0 + PolyForm Noncommercial

---

## 功能介绍

claude-mem 会自动捕获**Claude 在编码会话中所做的一切**：
- 工具调用（Read、Edit、Bash、Grep 等）
- 观察与发现
- 架构决策
- 文件修改

并在你重新连接项目时**智能注入**相关上下文。

**效果**：再也不用问"上次我们做了什么？"——Claude 会记得。

---

## 安装

### 通过插件市场安装（推荐）

```bash
# 添加插件市场
/plugin marketplace add thedotmack/claude-mem

# 安装插件
/plugin install claude-mem

# 重启 Claude Code
exit
claude
```

### 手动安装

```bash
# 需要 Bun 运行时（非 Node）——如未安装请先执行：
# curl -fsSL https://bun.sh/install | bash

# 克隆仓库
git clone https://github.com/thedotmack/claude-mem.git ~/.claude/plugins/claude-mem

# 安装依赖
cd ~/.claude/plugins/claude-mem
bun install

# 重启 Claude Code
```

> **依赖说明**：claude-mem 的 worker 运行在 [Bun](https://bun.sh) 上，而非 Node。如果你只安装了 Node，请先安装 Bun。缺少 Bun 时 worker 将无法启动，会话将不会被捕获（失败开放模式——Claude Code 仍可正常工作，只是没有记忆捕获功能）。

---

## 配置

### 默认配置

claude-mem **开箱即用**，内置合理的默认值：

```json
{
  "worker": {
    "port": 37777,
    "host": "127.0.0.1"
  },
  "storage": {
    "location": "~/.claude-mem/claude-mem.db",
    "backend": "sqlite"
  },
  "indexation": {
    "provider": "chroma",
    "embeddings": "claude"
  },
  "privacy": {
    "excludeTags": ["<private>", "</private>"]
  }
}
```

> ⚠️ **安全提示**：始终使用 `host: "127.0.0.1"`，切勿使用 `"0.0.0.0"`。`GET /api/settings` 接口会以明文返回 API 密钥——任何本地进程（浏览器扩展、npm 包）都可以读取到。仅绑定本地回环地址可减少攻击面，但在共享机器上并不能完全消除风险。

### 自定义配置

创建 `~/.claude-mem/config.json`：

```json
{
  "worker": {
    "port": 38888,
    "host": "localhost"
  },
  "compression": {
    "enabled": true,
    "summaryLength": "medium"
  },
  "privacy": {
    "excludeTags": ["<private>", "</private>", "<secret>", "</secret>"],
    "autoDetectSecrets": false
  },
  "storage": {
    "maxObservations": 10000,
    "retentionDays": 90
  }
}
```

**配置项说明**：

| 配置项 | 可选值 | 说明 |
|--------|--------|------|
| `compression.enabled` | true/false | 启用 AI 压缩（默认：true） |
| `compression.summaryLength` | short/medium/long | 摘要详细程度 |
| `privacy.autoDetectSecrets` | true/false | 自动检测 API 密钥（默认：false） |
| `storage.maxObservations` | 数字 | 最大存储观察条数 |
| `storage.retentionDays` | 数字 | N 天后自动删除 |

---

## 使用方法

### 自动捕获（默认行为）

**无需任何命令**——claude-mem 会自动：

1. **捕获**通过生命周期钩子触发的工具调用
2. **压缩**观察记录并生成 AI 摘要
3. **索引**至 Chroma 向量数据库
4. **注入**会话开始时的相关上下文

**会话示例流程**：

```
会话 1（第 1 天）：
用户："探索 auth 模块"
Claude：[读取 auth.service.ts、session.middleware.ts]
claude-mem：[捕获] "Auth 探索：JWT 验证，会话中间件"

会话 2（第 2 天）：
Claude：[自动注入]
"上次：探索了 auth 模块。
 文件：auth.service.ts、session.middleware.ts
 关键发现：validateToken() 中的 JWT 验证"
用户："将 auth 重构为使用 jose 库"
Claude：[已有上下文，无需重新读取]
```

---

### 可用技能

claude-mem 内置 5 个技能，通过 `/claude-mem:<技能名>` 调用：

| 技能 | 触发方式 | 使用场景 |
|------|----------|----------|
| `mem-search` | "我们是怎么修复 X 的？" | 通过自然语言搜索会话历史 |
| `smart-explore` | "探索 auth 模块" | 基于 AST 的代码导航（节省 token） |
| `make-plan` | "规划 Y 的重构" | 生成带文档发现的分阶段实施计划 |
| `do` | "执行计划" | 通过子智能体运行 `make-plan` 输出 |
| `timeline-report` | "展示我的历程" | 生成完整项目历史的叙述报告 |

### 自然语言搜索（`mem-search` 技能）

用自然语言搜索你的会话历史：

```bash
# 搜索特定主题
"搜索我的记忆中关于认证的决策"
"我们为修复支付 bug 改了哪些文件？"
"提醒我为什么选了 Zod 而不是 Yup"
"展示所有我们处理 API 的会话"
```

技能返回内容：
- 匹配会话及摘要
- 相关观察记录（类型：DISCOVERY / CHANGE / FEATURE / BUGFIX）
- 文件修改历史
- 架构决策

---

### Web 控制台

访问 `http://localhost:37777` 查看实时 UI：

```bash
# 打开控制台
open http://localhost:37777

# 功能：
# - 时间线视图（按时间顺序展示所有会话）
# - 自然语言搜索栏
# - 观察记录详情（工具调用 + 结果）
# - 会话统计（时长、工具使用量、修改文件数）
# - 导出/导入功能
```

**控制台模块**：

| 模块 | 说明 |
|------|------|
| **时间线** | 所有会话的时间顺序视图 |
| **搜索** | 自然语言查询界面 |
| **会话** | 带过滤器的列表视图 |
| **统计** | 使用分析与趋势 |
| **设置** | 隐私控制、存储管理 |

---

### 隐私控制

#### 使用 `<private>` 标签

```markdown
<!-- 在你的提示中使用 -->
请将数据库连接修改为：
<private>
Host: prod-db-123.aws.com
Username: admin
Password: super-secret-password
API Key: sk-1234567890abcdef
</private>

<!-- claude-mem 会排除 <private> 标签之间的内容 -->
```

#### 手动删除观察记录

```bash
# 删除特定观察记录
curl -X DELETE http://localhost:37777/api/observations/obs_123

# 清除某个会话的所有观察记录
curl -X DELETE http://localhost:37777/api/sessions/session_456/observations
```

#### 数据存储位置

```bash
# 数据库位置
~/.claude-mem/claude-mem.db

# Chroma 索引
~/.claude-mem/chroma/

# 查看数据库大小
du -sh ~/.claude-mem/
```

---

## 高级功能

### 渐进式披露

claude-mem 采用 3 层方式最小化 token 消耗：

```
第 1 层：搜索（50-100 个 token）
├─ 查询："查找认证相关工作"
├─ 返回：5 个会话摘要
│
第 2 层：时间线（500-1000 个 token）
├─ 查询："显示会话 abc123 的时间线"
├─ 返回：观察记录列表
│
第 3 层：详情（完整上下文）
└─ 查询："获取观察记录 obs_456 的详情"
    返回：完整工具调用 + 结果
```

**节省 token**：相比加载完整历史，约减少 10 倍。

---

### 无限模式（Beta）

用于长时间会话的实验性功能：

```bash
# 在配置中启用
{
  "experimental": {
    "endlessMode": true
  }
}
```

**声称效果**（未经独立验证）：
- 上下文减少约 95%
- 触及限制前可执行 20 倍以上的工具调用
- 激进压缩 + 智能摘要

⚠️ **注意**：Beta 功能，生产环境谨慎使用。

---

### 导出/导入

**导出会话历史**：

```bash
# 通过控制台
http://localhost:37777/export

# 通过 CLI
curl http://localhost:37777/api/export > claude-mem-backup.json
```

**在其他机器上导入**：

```bash
# 通过控制台
http://localhost:37777/import

# 通过 CLI
curl -X POST http://localhost:37777/api/import \
  -H "Content-Type: application/json" \
  -d @claude-mem-backup.json
```

---

## 费用说明

### API 压缩费用

| 使用强度 | 会话数/月 | 观察条数 | 预估费用（Claude Haiku） | 预估费用（Gemini Lite） |
|----------|-----------|----------|--------------------------|------------------------|
| **轻度** | 10-20 | 200-400 | $0.30-0.60 | ~$0.05 |
| **中度** | 50-80 | 1000-1600 | $1.50-2.40 | ~$0.20 |
| **重度** | 100-150 | 2000-3000 | $3.00-4.50 | ~$0.45 |
| **极重度** | ~400 次会话 | ~8000 | ~$102 | ~$14 |

**计算公式**：每 100 条观察约 $0.15（Claude Haiku）或约 $0.02（Gemini 2.5 Flash Lite）

**切换至 Gemini 可降低 86% 费用**：

```json
// ~/.claude-mem/settings.json
{
  "provider": "gemini",
  "model": "gemini-2.5-flash-lite",
  "auth_method": "cli"
}
```

> **Flash 与 Flash Lite 的权衡**：Gemini 2.5 Flash Lite 成本更低，但生成的摘要质量较弱。对大多数项目来说是可以接受的。对于上下文注入精度要求较高的复杂长期项目，建议使用 Gemini 2.5 Flash（非 Lite 版）。

**进一步降低费用（批处理）**：

```json
{
  "compression": {
    "batchSize": 50,
    "interval": "hourly"
  }
}
```

批量压缩（按小时执行）相比逐条压缩可减少 API 调用次数。

---

### 存储费用

**本地存储**（SQLite + Chroma）：

| 使用强度 | 存储量 |
|----------|--------|
| **轻度**（10 次会话/周） | 每月 10-20 MB |
| **中度**（50 次会话/周） | 每月 50-100 MB |
| **重度**（100 次会话/周） | 每月 100-200 MB |

**清理策略**：

```json
{
  "storage": {
    "retentionDays": 90,
    "autoCleanup": true
  }
}
```

---

## 排错

### 控制台无法加载

```bash
# 检查 worker 是否正在运行
curl http://localhost:37777/health
# 预期：{"status":"ok"}

# 重启 worker
claude-mem restart

# 查看日志
tail -f ~/.claude-mem/logs/worker.log
```

---

### API 费用过高

```bash
# 查看观察记录数量
curl http://localhost:37777/api/stats
# 返回：{"observations": 5234, "sessions": 123}

# 若观察记录过多：
# 1. 启用批处理
# 2. 增大压缩间隔
# 3. 缩短保留天数
```

---

### 记忆未被注入

```bash
# 验证索引状态
curl http://localhost:37777/api/index/status
# 预期：{"indexed": true, "observations": 1234}

# 手动触发重建索引
curl -X POST http://localhost:37777/api/index/rebuild
```

---

### 数据库损坏

```bash
# 先备份
cp ~/.claude-mem/claude-mem.db ~/.claude-mem/claude-mem.db.backup

# 重建索引
claude-mem index rebuild

# 若仍无法修复，重置（⚠️ 将丢失所有数据）
rm -rf ~/.claude-mem/
claude-mem init
```

---

## 许可证说明

### AGPL-3.0

**含义**：

- ✅ 个人使用免费
- ✅ 开源项目免费
- ⚠️ 网络使用 = 必须披露源代码
- ⚠️ 修改代码 = 必须披露源代码
- ❌ 不合规情况下不可用于闭源 SaaS

**商业使用**：

若用于商业产品：
1. 审查 AGPL-3.0 要求
2. 考虑法律合规性
3. 替代方案：联系作者获取商业授权

**PolyForm Noncommercial**（ragtime/ 目录）：
- 特定组件的独立许可证
- 商业限制更为严格

---

## 适用场景

### ✅ 推荐使用：

- 多会话项目（超过 1 周）
- 需要跨天/跨周记住决策
- 频繁重新连接同一项目
- 相比手动记录，更倾向自动捕获
- 需要 Web 控制台进行探索

### ❌ 不推荐使用：

- 一次性快速任务（不足 10 分钟）
- 极度敏感的数据（建议改用手动方式的 Serena）
- 未经 AGPL 合规审查的商业项目
- 需要跨机器同步（原生不支持）
- 预算有限（API 压缩费用 < $5/月）

---

## 与同类工具对比

| 工具 | 用途 | 捕获方式 | 查询方式 |
|------|------|----------|----------|
| **claude-mem** | 会话记忆 | 自动（钩子） | 自然语言 |
| **Serena** | 符号记忆 | 手动（`write_memory`） | 键值查找 |
| **grepai** | 语义搜索 | 不适用（仅搜索） | 语义 |
| **CLAUDE.md** | 项目上下文 | 手动（写文件） | Claude 启动时读取 |

**最佳组合使用**：
- claude-mem：自动会话捕获
- Serena：手动架构决策
- grepai：代码发现
- CLAUDE.md：项目规范

---

## 资源链接

**官方**：
- [GitHub 仓库](https://github.com/thedotmack/claude-mem)
- [文档](https://github.com/thedotmack/claude-mem/wiki)
- [发布说明](https://github.com/thedotmack/claude-mem/releases)

**指南**：
- [Corti.com：深度解析](https://corti.com/claude-mem-persistent-memory-for-ai-coding-assistants/)
- [yuv.ai：安装指南](https://yuv.ai/blog/claude-mem)
- [YouTube：5 分钟快速配置](https://www.youtube.com/watch?v=ryqpGVWRQxA)

**社区**：
- [GitHub Issues](https://github.com/thedotmack/claude-mem/issues)
- [GitHub Discussions](https://github.com/thedotmack/claude-mem/discussions)

---

**模板版本**：1.1.0
**最后更新**：2026-03-30
**指南版本**：3.38.1
